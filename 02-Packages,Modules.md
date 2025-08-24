### 1. Packages

A **package** is the fundamental unit of code organization in Go. It's a collection of Go source files (`.go` files) located in the same directory that are compiled together. Every Go file must belong to a package.

*   **Purpose:**
    *   **Organization:** To group related functions, types, and variables together, making code easier to manage. For example, the `fmt` package contains functions for formatting and printing output.
    *   **Encapsulation (Visibility):** To control which parts of your code are accessible to other packages. This is Go's version of "public" vs. "private".
        *   **Exported (Public):** If a name (like a function, type, or variable) starts with a **capital letter**, it is "exported." This means it can be accessed from any other package that imports this one.
        *   **Unexported (Private):** If a name starts with a **lowercase letter**, it is "unexported." It is only accessible within its own package.

*   **Declaration:** You declare a file's package at the very top of the source file using the `package` keyword.
    ```go
    // file: greetings/greetings.go
    package greetings // This file is part of the "greetings" package

    import "fmt"

    // Hello is an exported function because it starts with a capital 'H'.
    func Hello(name string) {
        fmt.Printf("Hello, %s!\n", name)
    }

    // secretMessage is an unexported variable, only usable inside the 'greetings' package.
    var secretMessage = "This is a secret."
    ```

*   **Importing:** To use code from another package, you use the `import` keyword.
    ```go
    // file: main.go
    package main

    import (
        "fmt"        // Importing a package from the Go standard library
        "greetings"  // Assuming 'greetings' is a package in our project
    )

    func main() {
        greetings.Hello("Alice") // We can call the exported 'Hello' function.
        // greetings.secretMessage // This would cause a compile error because secretMessage is not exported.
    }
    ```

### 2. Modules

A **module** is a collection of related Go packages that are versioned together as a single unit. Modules are Go's official dependency management system, introduced to solve the challenges of tracking and versioning the packages your project depends on.

*   **Key Concept:** A module is defined by a `go.mod` file at the root of the project's directory tree.
*   **The `go.mod` file:** This is the heart of a module. It's a text file that contains:
    *   **Module Path:** The unique name for the module, which defines its import path (e.g., `github.com/my-user/my-project`).
    *   **Go Version:** The version of Go the module was written for.
    *   **Dependencies (`require` block):** A list of all the other modules that your project depends on, along with their specific versions.

*   **Example `go.mod` file:**
    ```mod
    module example.com/my-app

    go 1.21

    require (
        github.com/gin-gonic/gin v1.9.1 // This project requires the Gin framework, version 1.9.1
        github.com/google/uuid v1.3.0   // It also requires the google/uuid package
    )
    ```

### 3. `go mod init`

This is the command you run to create a new module. It initializes a new project by creating the `go.mod` file in your current directory.

*   **How to use it:** You must provide the module path as an argument. This path is how other projects will import your code. It is a convention to use a path that corresponds to where your code is hosted, like a GitHub repository URL.

    ```bash
    # Navigate to your new project directory
    mkdir my-new-project
    cd my-new-project

    # Initialize the module
    # The module path is typically your repository URL
    go mod init github.com/your-username/my-new-project
    ```
*   **Result:** This command creates a `go.mod` file in the `my-new-project` directory with the following content:
    ```mod
    module github.com/your-username/my-new-project

    go 1.21 // Or whichever version of Go you are using
    ```
    Your project is now officially a Go module, and you can start adding code and managing dependencies. When you `go get` a new package or import one in your code and run `go mod tidy`, Go will automatically add it to the `require` block in your `go.mod` file.

### 4. `main.go` and the `main` package

In Go, there are two types of programs you can build: executables and libraries.

*   **Executable:** A program that you can run directly from your terminal.
*   **Library:** A package that is intended to be used by other programs (i.e., imported).

The Go compiler knows to build an executable if it finds a package named **`main`** that contains a function named **`main`**.

*   **`package main`:** This special package name tells the compiler that this is the entry point for an executable program.
*   **`func main()`:** This specific function is where the program's execution begins when you run the compiled file. It takes no arguments and returns no values.

A file is often named `main.go` by convention, but the filename doesn't matter to the compiler—only the `package main` and `func main()` declarations do.

*   **Example `main.go`:**
    ```go
    // This file must be declared as 'package main' to be an executable.
    package main

    import "fmt"

    // The program starts running here.
    func main() {
        fmt.Println("Hello, World!")
    }
    ```

### 5. `go build` and `go run` Commands

Both of these commands compile your Go code, but they serve different purposes.

#### `go build`

*   **What it does:** The `go build` command compiles the Go source files into a **standalone executable file**. It does *not* run the file.
*   **Output:** It creates a binary file in the current directory. On Windows, this will be `my-project.exe`; on Linux and macOS, it will be `my-project`. The name is taken from the module name or the directory name.
*   **When to use it:**
    *   When you want to create a distributable binary to deploy to a server or share with others.
    *   For the final step in a CI/CD pipeline before deploying your application.
    *   When you want to compile your code to check for errors without immediately running it.

*   **Example Usage:**
    ```bash
    # In your project directory containing main.go
    go build

    # You will now see an executable file
    ls
    # main.go   go.mod   my-project

    # You can now run this file directly
    ./my-project
    # Output: Hello, World!
    ```

#### `go run`

*   **What it does:** The `go run` command is a convenient shortcut that **compiles and runs** the specified Go program in one step.
*   **Output:** It compiles the code to a temporary executable in a temporary directory, runs that temporary executable, and then deletes it when the program finishes. **It does not leave a permanent binary file in your project directory.**
*   **When to use it:**
    *   During development for quickly testing changes to your code.
    *   For running small programs or scripts where you don't need to keep the compiled artifact.

*   **Example Usage:**
    ```bash
    # In your project directory
    # This command compiles and immediately runs main.go
    go run main.go
    # Output: Hello, World!

    # After the command finishes, check the directory contents
    ls
    # main.go   go.mod
    # Note: There is no executable file left behind.
    ```

---

**Summary Table:**

| Feature/Command | Description |
| :--- | :--- |
| **Package** | A directory of Go files compiled together. Used for organizing code and controlling visibility (exported/unexported). |
| **Module** | A collection of packages versioned together. Defined by a `go.mod` file, it's Go's dependency management system. |
| **`go mod init`** | Command to create a new module by generating a `go.mod` file. |
| **`package main`** | A special package declaration that marks the code as the entry point for an executable program. |
| **`go build`** | Compiles the code and creates a **permanent executable file**. Does not run the code. |
| **`go run`** | Compiles the code, runs the resulting program, and then **deletes the temporary executable**. |