### 1. Declaring Variables

In Go, a variable is a storage location with a specific type and an associated name. Go provides several ways to declare variables.

#### The `var` Keyword

This is the most explicit and fundamental way to declare a variable. It can be used both inside and outside of functions.

**1. Declare, then Assign (Zero Value)**
You can declare a variable without an initial value. In this case, Go will automatically assign it a "zero value."

*   `0` for numeric types (int, float)
*   `false` for `bool`
*   `""` (an empty string) for `string`
*   `nil` for pointers, interfaces, maps, slices, and channels.

```go
package main

import "fmt"

func main() {
    var score int      // Declared but not assigned.
    var name string    // Zero value is "".
    var isValid bool   // Zero value is false.

    fmt.Println("Default score:", score) // Output: Default score: 0
    
    score = 100 // Now we assign a value.
    fmt.Println("New score:", score) // Output: New score: 100
}
```

**2. Declare and Assign in One Line**
You can declare a variable and provide its initial value at the same time.

```go
var score int = 100
var name string = "Alice"
```

**3. Type Inference with `var`**
If you provide an initial value, Go can often infer the type, so you can omit it from the declaration.

```go
var score = 100          // Go infers this is an int
var name = "Alice"       // Go infers this is a string
var pi = 3.14          // Go infers this is a float64
```

#### The `:=` Short Variable Declaration

This is the most common and concise way to declare and initialize a variable. It's a shorthand that automatically infers the type.

*   **Rule:** This operator can **only be used inside functions**. It cannot be used at the package level (outside of a function).

```go
package main

import "fmt"

func main() {
    // The following three lines are equivalent to:
    // var score int = 100
    // var name string = "Alice"
    // var isReady bool = true
    
    score := 100
    name := "Alice"
    isReady := true

    fmt.Printf("Name: %s, Score: %d, Ready: %v\n", name, score, isReady)
}
```

You can also declare and initialize multiple variables at once with both `var` and `:=`.

```go
// Using var
var x, y, z = 1, 2, 3

// Using := inside a function
x, y, z := 1, 2, 3
```

### 2. Common Variable Types

Go has a rich set of built-in types.

*   **Boolean:**
    *   `bool`: Can only be `true` or `false`.

*   **Numeric:**
    *   **Integers (Signed):** `int`, `int8`, `int16`, `int32`, `int64`. The `int` type is the most common; its size (`32` or `64` bit) depends on the target system's architecture. It's generally the best choice unless you have a specific reason to use a sized integer.
    *   **Integers (Unsigned):** `uint`, `uint8` (also known as `byte`), `uint16`, `uint32`, `uint64`. These can only hold non-negative values.
    *   **Floating-Point:** `float32`, `float64`. The `float64` type is the default for floating-point values (like `3.14`) and is generally preferred for its higher precision.
    *   **Complex Numbers:** `complex64`, `complex128`.

*   **Strings:**
    *   `string`: A sequence of bytes. In Go, strings are **immutable**, meaning you cannot change their contents once they are created.

### 3. String Length: `len()` vs. `utf8.RuneCountInString()`

This is a very important and often misunderstood concept in Go. A Go string is a read-only slice of bytes, and it uses UTF-8 encoding by default.

#### `len()`

When you use the built-in `len()` function on a string, it returns the **number of bytes**, not the number of characters.

For standard ASCII characters (like a-z, 0-9), one character equals one byte, so `len()` will give you the result you expect. However, for many other languages and symbols, a single character can take up multiple bytes.

**Example:**

```go
package main

import "fmt"

func main() {
    // "hello" contains only ASCII characters.
    str1 := "hello"
    fmt.Printf("String: %s, Bytes: %d\n", str1, len(str1))
    // Output: String: hello, Bytes: 5

    // The character 'é' and the emoji '🌍' are multi-byte characters in UTF-8.
    str2 := "résumé" // 'é' is 2 bytes
    str3 := "Go🌍"   // '🌍' is 4 bytes

    fmt.Printf("String: %s, Bytes: %d\n", str2, len(str2))
    // Output: String: résumé, Bytes: 8 (r,e,s,u,m are 1 byte each. é is 2 bytes. 5 + 2 + 1 = 8)

    fmt.Printf("String: %s, Bytes: %d\n", str3, len(str3))
    // Output: String: Go🌍, Bytes: 6 (G=1, o=1, 🌍=4. 1 + 1 + 4 = 6)
}
```

#### `utf8.RuneCountInString()`

To correctly count the number of visible characters (called **runes** in Go), you must use the `RuneCountInString` function from the `unicode/utf8` package. A `rune` is Go's term for a single code point (a character).

**Example:**

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {
    str1 := "hello"
    str2 := "résumé"
    str3 := "Go🌍"

    // Using utf8.RuneCountInString to count characters (runes)
    fmt.Printf("String: %s, Characters: %d\n", str1, utf8.RuneCountInString(str1))
    // Output: String: hello, Characters: 5

    fmt.Printf("String: %s, Characters: %d\n", str2, utf8.RuneCountInString(str2))
    // Output: String: résumé, Characters: 7 (r, é, s, u, m, é)

    fmt.Printf("String: %s, Characters: %d\n", str3, utf8.RuneCountInString(str3))
    // Output: String: Go🌍, Characters: 3 (G, o, 🌍)
}
```

### 4. `const` vs. `var`

While both are used for declaring named values, they have a fundamental difference.

#### `var` (Variable)

*   **Mutable:** The value of a variable can be changed after it is declared.
*   **Runtime Value:** The value can be determined at runtime (e.g., from user input or a function call).
*   **Declaration:** Can be declared with or without an initial value. Can use the `:=` shortcut inside functions.

```go
var score = 50
score = 100 // This is valid. The value can be changed.
```

#### `const` (Constant)

*   **Immutable:** The value of a constant **cannot be changed** once it is declared. Trying to re-assign it will result in a compile-time error.
*   **Compile-time Value:** The value of a constant **must be known at compile time**. You cannot assign the result of a function call to a constant.
*   **Declaration:** Must be initialized when declared. Cannot use the `:=` shortcut.

```go
const Pi = 3.14159
// Pi = 3.14 // This would be a compile error: cannot assign to Pi

// The value must be known at compile-time.
// const randomValue = rand.Intn(10) // Compile error: rand.Intn(10) is called at runtime.```

You can also declare constants in a block for better organization.

```go
const (
    StatusOk      = 200
    StatusCreated = 201
    StatusNotFound  = 404
)
```

---

**Summary Table:**

| Feature | `var` (Variable) | `const` (Constant) |
| :--- | :--- | :--- |
| **Mutability** | Mutable (can be changed) | Immutable (cannot be changed) |
| **Value Time** | Can be determined at runtime | Must be known at compile-time |
| **Shorthand (`:=`)** | Yes, inside functions | No |
| **Zero Value** | Yes, if not initialized | No, must be initialized |
| **Typical Use** | Storing values that change during program execution (e.g., counters, user data). | Storing fixed values that never change (e.g., mathematical constants, configuration settings). |