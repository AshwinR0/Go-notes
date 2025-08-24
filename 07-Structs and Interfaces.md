### 1. Structs

A **struct** (short for structure) is a composite data type that groups together zero or more other data types (fields) under a single name. It's the primary way you create custom, complex data types in Go.

**Syntax:**
```go
type StructName struct {
    FieldName1 type1
    FieldName2 type2
    // ...
}
```

#### Ways of Creating Structs

Let's use a `Person` struct as an example.

```go
type Person struct {
    FirstName string
    LastName  string
    Age       int
}
```

**1. Using the `var` keyword (Zero-Valued)**
This creates an instance of the struct where all fields are initialized to their zero values (`""` for string, `0` for int).

```go
var p1 Person
p1.FirstName = "Alice"
p1.Age = 30
fmt.Println(p1) // Output: {Alice  30} (LastName is the zero value "")
```

**2. Using a Struct Literal**
This is the most common way. You can initialize it with or without specifying the field names.

```go
// With field names (recommended for clarity)
p2 := Person{
    FirstName: "Bob",
    LastName:  "Smith",
    Age:       42,
}

// Without field names (must be in order, brittle)
p3 := Person{"Charlie", "Jones", 25}
```

**3. Using the `new()` keyword**
The built-in `new()` function allocates memory for a new item of a given type, initializes it to its zero value, and returns a **pointer** to it.

```go
p4 := new(Person) // p4 is of type *Person (a pointer to a Person)
p4.FirstName = "Diana" // Go automatically dereferences the pointer for you
fmt.Println(*p4) // Output: {Diana  0}
```

**4. Pointer to a Struct Literal**
This is the idiomatic way to create a pointer to a new struct instance.

```go
p5 := &Person{
    FirstName: "Eve",
    LastName:  "Williams",
    Age:       35,
}
// p5 is also of type *Person
```

#### Creating a Struct with Another Struct Inside (Composition)

You can embed one struct within another to build more complex types. This is Go's primary way to achieve composition.

```go
type Address struct {
    Street string
    City   string
}

type Employee struct {
    Name    string
    Position string
    Location Address // The Address struct is a field inside Employee
}

func main() {
    emp := Employee{
        Name:    "Frank",
        Position: "Engineer",
        Location: Address{ // Initialize the nested struct
            Street: "123 Go Lane",
            City:   "Gopherville",
        },
    }

    fmt.Println(emp.Name)             // Output: Frank
    fmt.Println(emp.Location.City)    // Access nested fields with dot notation
}
```

#### Anonymous Structs

An anonymous struct is a struct that is defined inline without a formal `type` name. It's useful for short-lived data structures where a full type definition would be overkill, such as for unmarshalling a specific piece of JSON.

```go
func main() {
    // Define and initialize an anonymous struct in one go
    point := struct {
        X int
        Y int
    }{
        X: 10,
        Y: 20,
    }

    fmt.Printf("Point: %+v\n", point) // Output: Point: {X:10 Y:20}
}
```

### 2. Struct Methods

A **method** is a function that has a special "receiver" argument. The receiver connects the function to a specific type, like a struct. This is how you define behaviors for your custom types.

#### Method Signature

The syntax is similar to a function, but you place the receiver in parentheses before the function name.

```go
func (receiverName ReceiverType) MethodName(parameters) returnTypes {
    // Method body
}
```
*   `receiverName`: A name to refer to the instance of the struct inside the method (e.g., `p` for `Person`).
*   `ReceiverType`: The type the method is attached to (e.g., `Person` or `*Person`).

**Example:**
```go
type Rectangle struct {
    Width  float64
    Height float64
}

// This is a method attached to the Rectangle struct.
// 'r' is the receiver, a copy of the Rectangle instance.
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    fmt.Println("Area:", rect.Area()) // Call the method using dot notation.
}
```

#### Pointer vs. Value Receivers

This is a critical distinction.

*   **Value Receiver `(r Rectangle)`:** The method operates on a **copy** of the struct. Any modifications made to the receiver inside the method **will not** affect the original struct. Use this for methods that don't need to change the struct's state.

*   **Pointer Receiver `(r *Rectangle)`:** The method operates on a **pointer** to the original struct. Modifications made inside the method **will** affect the original struct. Use this when you need to mutate the struct's data.

```go
type Person struct {
    Name string
    Age  int
}

// Value receiver: operates on a copy.
func (p Person) Greet() {
    fmt.Printf("Hello, my name is %s and I am %d years old.\n", p.Name, p.Age)
}

// Pointer receiver: can modify the original struct.
func (p *Person) HaveBirthday() {
    p.Age++ // This modifies the original Person's Age field.
}

func main() {
    p := Person{Name: "Alice", Age: 30}
    p.Greet() // Output: Hello, my name is Alice and I am 30 years old.

    p.HaveBirthday() // The Go compiler automatically converts 'p' to '&p' here.
    p.Greet() // Output: Hello, my name is Alice and I am 31 years old.
}
```
**Guideline:** If you're unsure, use a pointer receiver. It's more efficient for large structs and is necessary for any method that needs to modify the receiver.

### 3. Interfaces

An **interface** is a custom type that defines a set of method signatures. It specifies *what methods a type should have*, but not *how those methods are implemented*. It is a contract for **behavior**.

**Syntax:**
```go
type InterfaceName interface {
    MethodName1(params) returnTypes
    MethodName2(params) returnTypes
    // ...
}
```

#### Implicit Implementation

This is the most powerful feature of Go interfaces. A type is said to **satisfy** an interface if it defines all the methods in that interface's signature. There is no `implements` keyword. This "duck typing" makes Go's interfaces very flexible and decoupled.

**Complete Example:**

1.  **Define the Interface (the contract):**
    We define a `Shape` behavior. Anything that is a `Shape` must have a method called `Area()` that returns a `float64`.
    ```go
    type Shape interface {
        Area() float64
    }
    ```

2.  **Define Concrete Types (the implementers):**
    We create two different structs, `Rectangle` and `Circle`.
    ```go
    type Rectangle struct {
        Width, Height float64
    }

    type Circle struct {
        Radius float64
    }
    ```

3.  **Implement the Methods:**
    We implement the `Area()` method for *both* types. Because they now both have this method, they both *implicitly satisfy* the `Shape` interface.
    ```go
    func (r Rectangle) Area() float64 {
        return r.Width * r.Height
    }

    func (c Circle) Area() float64 {
        return math.Pi * c.Radius * c.Radius
    }
    ```

4.  **Use the Interface (Polymorphism):**
    We can now write a function that accepts the `Shape` interface as its parameter. This function doesn't care if it gets a `Rectangle` or a `Circle`; it only cares that whatever it gets has an `Area()` method.
    ```go
    // This function can take any type that satisfies the Shape interface.
    func PrintShapeArea(s Shape) {
        fmt.Printf("The area of this shape is: %0.2f\n", s.Area())
    }
    ```

5.  **Putting it all together:**
    ```go
    import ("fmt"; "math")

    func main() {
        rect := Rectangle{Width: 10, Height: 5}
        circ := Circle{Radius: 4}

        // We can pass both a Rectangle and a Circle to the same function,
        // because both satisfy the Shape interface.
        PrintShapeArea(rect)
        PrintShapeArea(circ)
    }
    ```
    **Output:**
    ```
    The area of this shape is: 50.00
    The area of this shape is: 50.27
    ```