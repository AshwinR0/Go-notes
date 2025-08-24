### **Arrays**

An array is a numbered sequence of elements of a single, specific type. They form the basis for slices but are less commonly used directly in Go code due to their inflexibility.

**Core Properties:**

1.  **Fixed Length:** The size of an array is part of its type. An array of type `[5]int` is a different and incompatible type from an array of type `[10]int`. You cannot change the size of an array after it's created.
2.  **Same Type:** All elements in an array must be of the same type (e.g., you can't mix `int` and `string` in the same array).
3.  **Indexable:** Elements are accessed by a zero-based index, from `0` to `length - 1`.
4.  **Contiguous in Memory:** The elements of an array are stored in a single, unbroken block of memory, which makes them very efficient to access.

#### Ways of Array Initialization

```go
package main
import "fmt"

func main() {
    // 1. Declare a zero-valued array.
    // All elements are initialized to their zero value (0 for int).
    var scores [5]int
    fmt.Println("Zero-valued:", scores) // Output: [0 0 0 0 0]
    scores[0] = 95 // Assign a value

    // 2. Declare and initialize using an array literal.
    primes := [6]int{2, 3, 5, 7, 11, 13}
    fmt.Println("Literal:", primes) // Output: [2 3 5 7 11 13]

    // 3. Let the compiler infer the length using the '...' syntax.
    // The compiler counts the number of elements in the literal.
    // This is the most common and convenient way to initialize an array.
    names := [...]string{"Alice", "Bob", "Charlie"}
    fmt.Println("Inferred length:", names, "Length:", len(names)) // Output: [Alice Bob Charlie] Length: 3

    // 4. Initialize specific elements by index.
    // Unspecified elements are zero-valued.
    grades := [4]int{0: 95, 3: 88}
    fmt.Println("Indexed init:", grades) // Output: [95 0 0 88]
}
```

---

### **Slices**

A slice is a flexible, dynamic view into the elements of an underlying array. Slices are far more common and powerful than arrays in Go. A slice itself doesn't store any data; it just *describes* a section of an underlying array.

A slice header consists of three parts:
1.  **Pointer:** Points to the first element of the underlying array that is accessible through the slice.
2.  **Length:** The number of elements the slice contains.
3.  **Capacity:** The number of elements in the underlying array, starting from the element the slice points to.

#### `len()` and `cap()` functions

*   `len(s)`: Returns the **length** of the slice `s` (the number of elements it contains).
*   `cap(s)`: Returns the **capacity** of the slice `s` (the maximum number of elements it can hold before a new underlying array must be allocated).

#### Ways of Making a Slice

```go
package main
import "fmt"

func main() {
    // 1. Using a slice literal (most common).
    // This creates an array behind the scenes and returns a slice that refers to it.
    letters := []string{"a", "b", "c", "d"}
    fmt.Printf("Literal: %v, len: %d, cap: %d\n", letters, len(letters), cap(letters))

    // 2. Using the make() function (for preallocation).
    // Creates a slice of strings with a length of 3 and a capacity of 5.
    // The slice is initialized with zero values ("").
    names := make([]string, 3, 5)
    fmt.Printf("Made slice: %v, len: %d, cap: %d\n", names, len(names), cap(names))
    names[0] = "Frodo"
    names[1] = "Sam"
    names[2] = "Pippin"
    
    // 3. Slicing an existing array or slice.
    // Syntax: a[low:high]
    // 'low' is inclusive, 'high' is exclusive.
    primes := [6]int{2, 3, 5, 7, 11, 13}
    var s []int = primes[1:4] // Creates a slice from index 1 up to (but not including) 4.
    fmt.Printf("Sliced: %v, len: %d, cap: %d\n", s, len(s), cap(s)) 
    // Output: [3 5 7], len: 3, cap: 5 (capacity is from index 1 to the end of the array)
}
```

#### Slice Operations

The most important slice operation is `append`. It adds elements to the end of a slice. If the underlying array has enough capacity, the slice's length is extended. If not, a new, larger array is allocated, and all existing elements are copied over.

```go
var mySlice []int // A nil slice
mySlice = append(mySlice, 10) // Appends 10
mySlice = append(mySlice, 20, 30) // Appends multiple elements
fmt.Println(mySlice) // Output: [10 20 30]
```

---

### **Maps**

A map is an unordered collection of key-value pairs. It's Go's implementation of a hash table. The keys must be of a "comparable" type (like `int`, `string`, `bool`, pointers), and all keys must be of the same type.

#### Ways of Making a Map

```go
package main
import "fmt"

func main() {
    // 1. Using the make() function (creates an empty map).
    ages := make(map[string]int)
    ages["Alice"] = 30
    
    // 2. Using a map literal.
    scores := map[string]int{
        "Bob":     88,
        "Charlie": 95,
    }
    fmt.Println(scores)
}
```

#### Map Operations

```go
scores := make(map[string]int)

// 1. Add or Update an element
scores["Alice"] = 92
scores["Bob"] = 75
scores["Alice"] = 95 // This updates the existing value for key "Alice"

// 2. Retrieve an element
bobsScore := scores["Bob"]
fmt.Println(bobsScore) // Output: 75

// 3. Delete an element
delete(scores, "Bob")
fmt.Println(scores) // Output: map[Alice:95]
```

#### Map Optional Return Value ("comma ok" idiom)

When you access a key in a map, it will return the value for that key. If the key doesn't exist, it returns the zero value for the value type (e.g., `0` for `int`, `""` for `string`). This can be ambiguous. To solve this, a map access can optionally return a second boolean value that tells you if the key existed.

```go
scores := map[string]int{"Alice": 95}

// Check for a key that exists
val, ok := scores["Alice"]
fmt.Printf("Alice's score: %d, Exists: %v\n", val, ok) // Output: Alice's score: 95, Exists: true

// Check for a key that does NOT exist
val, ok = scores["Bob"]
fmt.Printf("Bob's score: %d, Exists: %v\n", val, ok) // Output: Bob's score: 0, Exists: false
// 'val' is 0 (the zero value for int), and 'ok' is false.

// This idiom is often used directly in an 'if' statement
if val, ok := scores["Bob"]; ok {
    fmt.Println("Bob's score is", val)
} else {
    fmt.Println("Bob is not in the map.")
}
```

---

### **Iteration and Loops**

Go has only one looping construct: the `for` loop, but it has several forms.

**1. The "While" Loop:**
```go
n := 0
for n < 5 {
    fmt.Println(n)
    n++
}
```

**2. The Complete `for` Loop (C-style):**
```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

#### Iterating with `range`

The `range` keyword is the idiomatic way to loop over arrays, slices, and maps.

**Iterating over a Slice or Array:** `range` returns the index and a copy of the value at that index.```go
names := []string{"Alice", "Bob", "Charlie"}
for index, value := range names {
    fmt.Printf("Index: %d, Value: %s\n", index, value)
}

// If you don't need the index, use the blank identifier '_'.
for _, value := range names {
    fmt.Println("Value:", value)
}
```

**Iterating over a Map:** `range` returns the key and the value. The order of iteration over a map is **not guaranteed**.
```go
scores := map[string]int{"Alice": 95, "Bob": 75}
for key, value := range scores {
    fmt.Printf("Key: %s, Value: %d\n", key, value)
}
```

---

### **Difference Between Preallocation and Without Preallocation**

This is a crucial performance concept for slices and maps.

#### Without Preallocation

When you create a slice or map without specifying a size, it starts with a small initial capacity. As you add elements (e.g., using `append`), if the capacity is exceeded, the Go runtime must perform an expensive operation:
1.  Allocate a new, larger block of memory (a new underlying array).
2.  Copy all the existing elements from the old block to the new one.
3.  The old block is then left for the garbage collector to clean up.

This process of reallocating and copying can happen many times in a loop, significantly slowing down your program.

```go
// WITHOUT preallocation
var numbers []int
for i := 0; i < 1000; i++ {
    // This append might cause many reallocations and copies.
    numbers = append(numbers, i)
}
```

#### With Preallocation

If you know how many elements you are going to store (or have a good estimate), you can **preallocate** the necessary memory upfront using `make`.

This creates a slice or map with sufficient initial capacity. Now, when you add elements, no reallocations are needed until you exceed that initial capacity. This is much more efficient.

```go
// WITH preallocation
// We know we need 1000 elements, so we make a slice with that length.
numbers := make([]int, 1000)
for i := 0; i < 1000; i++ {
    // No reallocations happen here. We just assign to existing slots.
    numbers[i] = i
}

// If you want an empty slice but with capacity:
numbersWithCap := make([]int, 0, 1000)
for i := 0; i < 1000; i++ {
    // Appending will be fast because capacity is already there.
    numbersWithCap = append(numbersWithCap, i)
}
```

**Rule of Thumb:** If you know the size of the slice or map you need to build, **always preallocate** using `make` to improve performance.