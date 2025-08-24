Building a well-structured API is a core skill in Go.

We'll use the popular `chi` router for these examples because it's lightweight, fast, and works seamlessly with the standard `net/http` library, making it excellent for learning.

---

### 1. The Foundation: Database Connection & Main Setup

Before you can serve requests, your application needs to start up and connect to its dependencies, like a database.

#### Database Connection (`database.go`)

It's good practice to encapsulate your database logic. A function should be responsible for creating and configuring the database connection pool. Go's `database/sql` package manages this pool for you automatically.

*   **Connection String:** Contains all the information needed to connect (user, password, host, database name). It's best to load this from configuration (e.g., environment variables) rather than hardcoding it.
*   **`sql.Open`:** This function opens a connection pool but doesn't actually create a connection or check if the credentials are valid.
*   **`db.Ping`:** This is used to verify that a connection to the database can be successfully established.

**Example Code:**
```go
package main // Assuming this is in the same package for simplicity

import (
	"database/sql"
	"fmt"
	"log"
	_ "github.com/lib/pq" // PostgreSQL driver, the blank import registers the driver
)

func ConnectDB() (*sql.DB, error) {
	// In a real app, get this from config/env vars
	connStr := "user=postgres password=secret dbname=mydb host=localhost sslmode=disable"

	db, err := sql.Open("postgres", connStr)
	if err != nil {
		return nil, fmt.Errorf("failed to open database connection: %w", err)
	}

	// Ping the database to verify the connection is alive
	if err := db.Ping(); err != nil {
		db.Close() // Clean up on failure
		return nil, fmt.Errorf("failed to ping database: %w", err)
	}

	log.Println("Successfully connected to the database!")
	return db, nil
}
```

---

### 2. Setting the Routes (The Router)

The router is the traffic cop of your API. It inspects the incoming request's URL and HTTP method (GET, POST, etc.) and directs it to the correct handler function.

*   **`chi.NewRouter()`:** Creates a new router instance.
*   **`r.Use(...)`:** Applies middleware to the router. Middleware attached here will run for *every single request*.
*   **`r.Get("/", ...)`:** Maps an HTTP GET request for the path `/` to a specific handler.
*   **`r.Group(...)`:** A powerful feature for grouping routes that share common properties, like a URL prefix or a set of middleware.

**Example Code:**
```go
// Inside your main function or a dedicated routes function

// ... create router
r := chi.NewRouter()

// Apply global middleware (like the logger)
r.Use(LoggerMiddleware)

// --- Public Routes ---
// These routes are accessible to anyone.
r.Get("/", publicHandler)
r.Get("/health", healthCheckHandler)

// --- Private Routes (Group) ---
// We create a group for all routes that require authentication.
r.Group(func(r chi.Router) {
    // Apply the authentication middleware ONLY to this group.
    r.Use(AuthMiddleware)

    // All routes defined inside this group are now protected.
    r.Get("/private/data", privateDataHandler)
    r.Post("/private/item", createItemHandler)
})
```

---

### 3. Middleware (The Guards and Helpers)

**Middleware** is a function that sits between the router and your final handler. It can inspect, modify, or even halt a request before it reaches its destination. It's the perfect place for cross-cutting concerns.

The standard signature for chi middleware is `func(next http.Handler) http.Handler`.

#### Logger Middleware

This middleware logs details about every incoming request. It's invaluable for debugging and monitoring.

**How it works:**
1.  It gets the `next` handler in the chain.
2.  It records the start time.
3.  It calls `next.ServeHTTP(w, r)` to pass control down the line.
4.  Once the handler finishes, it calculates the duration and logs everything.

**Example Code:**
```go
func LoggerMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		
		// Let the next handler in the chain execute
		next.ServeHTTP(w, r)
		
		// After the handler has finished, log the request details
		log.Printf(
			"%s %s %s",
			r.Method,
			r.RequestURI,
			time.Since(start),
		)
	})
}
```

---

### 4. Authorization Middleware (Private vs. Public)

This middleware is responsible for protecting routes. It checks for proof of identity (like an API key or a token) and denies access if it's missing or invalid.

**How it works:**
1.  It checks for an `Authorization` header in the request.
2.  It validates the token/key. (In this simple example, we'll just check for a hardcoded secret.)
3.  **If valid:** It calls `next.ServeHTTP(w, r)` to allow the request to proceed to the actual handler.
4.  **If invalid:** It writes an error response (e.g., `401 Unauthorized`) and **does not** call `next`. This stops the request chain cold.

**Example Code:**
```go
const hardcodedAPIKey = "super-secret-key"

func AuthMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Get the API key from the request header
		apiKey := r.Header.Get("Authorization")

		// Validate the key
		if apiKey != hardcodedAPIKey {
			// Write an error response and return immediately
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}

		// If the key is valid, call the next handler
		next.ServeHTTP(w, r)
	})
}
```

---

### 5. Putting It All Together (`main.go`)

Now we combine all the pieces into a runnable application.

```go
package main

import (
	"database/sql"
	"log"
	"net/http"
	"time"

	"github.com/go-chi/chi/v5"
	_ "github.com/lib/pq"
)

// ... (paste ConnectDB, LoggerMiddleware, and AuthMiddleware code here) ...

// --- Handlers ---
func publicHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("This is a public endpoint."))
}

func healthCheckHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("OK"))
}

func privateDataHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("This is secret data, only for authorized users."))
}

func createItemHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Item created successfully."))
}

// --- Main Application ---
func main() {
	// Connect to the database
	db, err := ConnectDB()
	if err != nil {
		log.Fatalf("Could not connect to the database: %v", err)
	}
	defer db.Close()
	// You can now pass 'db' to your handlers that need it.

	// Create the router
	r := chi.NewRouter()

	// Apply global middleware
	r.Use(LoggerMiddleware)

	// Define public routes
	r.Get("/", publicHandler)
	r.Get("/health", healthCheckHandler)

	// Define private routes using a group and auth middleware
	r.Group(func(r chi.Router) {
		r.Use(AuthMiddleware)
		r.Get("/private/data", privateDataHandler)
		r.Post("/private/item", createItemHandler)
	})
	
	port := ":8080"
	log.Printf("Starting server on port %s", port)
	
	// Start the server
	if err := http.ListenAndServe(port, r); err != nil {
		log.Fatalf("Could not start server: %v", err)
	}
}
```

**How to Run and Test:**

1.  Save the code.
2.  Run `go mod init myapi` and `go mod tidy`.
3.  Run `go run .`.
4.  Use a tool like `curl` to test your endpoints:
    *   **Public:** `curl http://localhost:8080/` -> `This is a public endpoint.`
    *   **Private (No Auth):** `curl http://localhost:8080/private/data` -> `Unauthorized`
    *   **Private (With Auth):** `curl -H "Authorization: super-secret-key" http://localhost:8080/private/data` -> `This is secret data...`