### **Functions in Go**

A function is a reusable block of code that performs a specific task. In Go, functions are fundamental building blocks. The basic syntax is:

```go
func functionName(parameter1 type, parameter2 type) returnType {
    // function body
    return value
}
```

#### 1. Passing Parameters

Go is strictly a **pass-by-value** language. This means that when you pass a variable to a function, the function receives a **copy** of the variable's value, not a reference to the original variable itself.

*   **Passing Basic Types (int, string, bool, struct):** Any changes made to the parameter inside the function will **not** affect the original variable outside the function.

    ```go
    package main
    import "fmt"

    func modifyValue(val int) {
        val = 100 // This changes the copy, not the original 'number'.
        fmt.Println("Inside function:", val) // Output: Inside function: 100
    }

    func main() {
        number := 10
        modifyValue(number)
        fmt.Println("Outside function:", number) // Output: Outside function: 10
    }
    ```

*   **Simulating "Pass-by-Reference" with Pointers:** To allow a function to modify the original variable, you must pass a **pointer** to that variable (its memory address). The function then receives a copy of the memory address, which it can use to access and modify the original value.

    ```go
    package main
    import "fmt"

    // The function now accepts a pointer to an int (*int).
    func modifyValueWithPointer(val *int) {
        *val = 100 // The '*' dereferences the pointer, modifying the original value.
        fmt.Println("Inside function (address):", val)
        fmt.Println("Inside function (value):", *val)
    }

    func main() {
        number := 10
        fmt.Println("Original value:", number)
        // We pass the memory address of 'number' using the '&' operator.
        modifyValueWithPointer(&number)
        fmt.Println("Value after function call:", number) // Output: Value after function call: 100
    }
    ```
    *Note on Slices and Maps:* Slices and maps are reference types. When you pass them to a function, a copy of the slice/map header is made, but this header still points to the same underlying data array. Therefore, if a function modifies the *elements* of a slice or map, the changes will be visible outside the function.

#### 2. Returning Values

A function can return a single value. The type of the return value is specified after the parameter list.

```go
package main
import "fmt"

// This function takes two integers and returns their sum as an integer.
func add(a int, b int) int {
    return a + b
}

func main() {
    result := add(5, 3)
    fmt.Println("Result:", result) // Output: Result: 8
}
```

#### 3. Returning Multiple Values

A key feature of Go is the ability for functions to return multiple values. This is incredibly useful for returning a result along with a status or an error. The return types are listed in parentheses.

```go
package main
import "fmt"

// This function returns two values: an integer result and a boolean status.
func divide(a int, b int) (int, bool) {
    if b == 0 {
        return 0, false // Cannot divide by zero, return 0 and a 'false' status.
    }
    result := a / b
    return result, true // Return the result and a 'true' status.
}

func main() {
    // We can assign the returned values to two separate variables.
    res, ok := divide(10, 2)
    if ok {
        fmt.Println("Result:", res) // Output: Result: 5
    }

    res, ok = divide(10, 0)
    if !ok {
        fmt.Println("Division failed!") // Output: Division failed!
    }

    // If you only care about one of the return values, you can discard
    // the others using the blank identifier '_'.
    justTheResult, _ := divide(12, 4)
    fmt.Println("Just the result:", justTheResult)
}
```

#### 4. The `error` Type

The most common use for multiple return values in Go is the idiomatic error handling pattern. The `error` type is a built-in interface in Go. By convention, functions that can fail return the `error` as their last return value.

*   If the function succeeds, it returns the result and a `nil` error.
*   If the function fails, it returns a zero value for the result and a non-`nil` error object describing what went wrong.

```go
package main
import (
	"errors"
	"fmt"
)

// This function returns a string and an error.
func greet(name string) (string, error) {
    if name == "" {
        // If the name is empty, create a new error.
        return "", errors.New("cannot greet an empty name")
    }
    greeting := fmt.Sprintf("Hello, %s!", name)
    return greeting, nil // Success! Return the greeting and a nil error.
}

func main() {
    // Standard Go error handling pattern
    msg, err := greet("Alice")
    if err != nil {
        // This block runs if something went wrong.
        fmt.Println("An error occurred:", err)
    } else {
        // This block runs on success.
        fmt.Println(msg) // Output: Hello, Alice!
    }
    
    msg, err = greet("")
    if err != nil {
        fmt.Println("An error occurred:", err) // Output: An error occurred: cannot greet an empty name
    } else {
        fmt.Println(msg)
    }
}
```

---

### **Control Structures**

Control structures allow you to direct the flow of your program's execution based on certain conditions.

#### 1. `if`, `else if`, `else`

The `if` statement executes a block of code if a condition is true. Go's `if` statement is similar to other languages, but it **does not use parentheses** around the condition.

**Basic Syntax:**
```go
if condition {
    // code to execute if condition is true
} else if anotherCondition {
    // ...
} else {
    // ...
}
```

**Statement Initializer:**
A powerful Go idiom is to include a short initialization statement before the condition, separated by a semicolon. Variables declared in this initializer are **scoped only to the `if/else` block**. This is very commonly used with the error handling pattern.

```go
package main
import "fmt"

func main() {
    score := 85

    if score > 90 {
        fmt.Println("Grade: A")
    } else if score > 75 {
        fmt.Println("Grade: B") // This will be printed.
    } else {
        fmt.Println("Grade: C or below")
    }
    
    // Example with the statement initializer
    // 'val' is only accessible inside this if/else block.
    if val := score / 10; val >= 8 {
        fmt.Printf("Excellent score! (Value: %d)\n", val)
    } else {
        fmt.Printf("Good score. (Value: %d)\n", val)
    }
    // fmt.Println(val) // This would be a compile error: 'val' is undefined here.
}
```

#### 2. `switch`

The `switch` statement is a more expressive way to write a multi-way conditional, especially when compared to a long chain of `if-else if` statements.

**Key Features in Go:**
*   **Implicit `break`:** Unlike C or Java, a `case` in Go automatically breaks. You do not need to add a `break` statement at the end of each case.
*   **`fallthrough`:** If you explicitly want execution to continue to the next case, you can use the `fallthrough` keyword. This is rarely used but is available when needed.
*   **Expressionless `switch`:** You can omit the expression after the `switch` keyword. This makes it equivalent to `switch true` and is a clean way to write a complex `if-else` chain where cases have different conditions.

**Standard `switch`:**```go
package main
import "fmt"

func main() {
    day := "Wednesday"

    switch day {
    case "Monday":
        fmt.Println("Start of the work week.")
    case "Tuesday", "Wednesday", "Thursday": // You can list multiple conditions
        fmt.Println("Mid-week.")
    case "Friday":
        fmt.Println("Almost there!")
    default:
        fmt.Println("It's the weekend!")
    }
}
```

**Expressionless `switch`:**
This is a very idiomatic and readable way to structure complex conditional logic.

```go
package main
import "fmt"

func main() {
    score := 77

    switch { // This is equivalent to 'switch true'
    case score >= 90:
        fmt.Println("Grade: A")
    case score >= 80:
        fmt.Println("Grade: B")
    case score >= 70:
        fmt.Println("Grade: C") // This case is matched first and execution stops.
    default:
        fmt.Println("Grade: D or below")
    }
}
```