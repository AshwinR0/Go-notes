### 1. Strings, Runes, and Bytes

Understanding the distinction between these three concepts is fundamental to working with text correctly in Go.

#### What is a String?

In Go, a `string` is an **immutable sequence of bytes**. This is a critical definition.

*   **Immutable:** Once a string is created, you cannot change its contents. Operations that appear to modify a string, like concatenation (`s1 + s2`), actually create a completely new string in memory.
*   **Sequence of bytes:** A string is, at its core, just a slice of bytes (`[]byte`). The Go runtime adds a convention that this slice of bytes should, whenever possible, contain valid UTF-8 encoded text.

#### What is a Byte?

A `byte` is simply an alias for the `uint8` type. It represents an 8-bit unsigned integer, which is the standard unit of data in computing. When you see `[]byte`, it means a "slice of bytes," which is the most common way to handle raw binary data.

#### What is a Rune?

A `rune` is an alias for the `int32` type. It is Go's concept for a single **Unicode code point**, which is the programming term for a "character." A rune can represent everything from the letter 'A' to the Chinese character '好' to an emoji like '🚀'.

Since a `rune` is an `int32`, it can represent any of the over 140,000 Unicode code points in existence.

#### The Relationship: An Analogy

Think of a multi-language book:
*   **String:** The entire book (`"Hello, résumé, Go🚀"`).
*   **Bytes:** The raw ink on the page. Some "letters" require more ink than others. This is the underlying data stored in memory.
*   **Runes:** The individual letters and symbols you actually read ('H', 'e', 'l', 'l', 'o', ',', ' ', 'r', 'é', 's', 'u', 'm', 'é', ',', ' ', 'G', 'o', '🚀').

**Code Example:**

The way you iterate over a string reveals this difference. A standard `for` loop sees the bytes. A `for...range` loop intelligently decodes the UTF-8 bytes and sees the runes.

```go
package main

import "fmt"

func main() {
    s := "Go🚀"

    fmt.Printf("String: %s\n", s)
    fmt.Printf("Length in bytes (len()): %d\n", len(s)) // G=1 byte, o=1 byte, 🚀=4 bytes

    fmt.Println("\nIterating by BYTES (standard for loop):")
    for i := 0; i < len(s); i++ {
        // This prints the raw byte values in hexadecimal format.
        fmt.Printf("Index %d: %x\n", i, s[i])
    }
    // Notice how it takes 4 bytes to represent the emoji.

    fmt.Println("\nIterating by RUNES (for...range loop):")
    // 'range' on a string decodes one rune at a time.
    // 'i' is the starting byte index of the rune.
    // 'r' is the rune itself.
    for i, r := range s {
        fmt.Printf("Byte Index %d: Rune '%c' (Unicode: %U)\n", i, r, r)
    }
}
```
**Output:**
```
String: Go🚀
Length in bytes (len()): 6

Iterating by BYTES (standard for loop):
Index 0: 47
Index 1: 6f
Index 2: f0
Index 3: 9f
Index 4: 9a
Index 5: 80

Iterating by RUNES (for...range loop):
Byte Index 0: Rune 'G' (Unicode: U+0047)
Byte Index 1: Rune 'o' (Unicode: U+006F)
Byte Index 2: Rune '🚀' (Unicode: U+1F680)
```

### 2. UTF-8 Encoding

UTF-8 is the encoding scheme that Go uses to translate between runes (characters) and bytes (the data in the string). Its most important feature is that it is a **variable-width encoding**.

*   **1 Byte:** For all standard ASCII characters (English letters, numbers, common punctuation). This makes UTF-8 backward compatible with ASCII.
*   **2 Bytes:** For Latin letters with diacritics (like `é`, `ü`), Greek, Cyrillic, etc.
*   **3 Bytes:** For most other common characters, including Chinese, Japanese, and Korean.
*   **4 Bytes:** For everything else, including most emojis and historical scripts.

This is precisely why `len()` and the actual character count can be different. `len()` always tells you the total number of bytes used, while `utf8.RuneCountInString()` tells you the number of runes.

### 3. `strings.Builder`

This is a highly efficient tool for building strings.

#### The Problem: String Immutability and Concatenation

Because strings are immutable, every time you concatenate them using the `+` or `+=` operator, Go must:
1.  Allocate a new block of memory large enough for the combined string.
2.  Copy the bytes from the first string into the new block.
3.  Copy the bytes from the second string into the new block.

Doing this repeatedly in a loop is extremely inefficient, as it creates many temporary string objects that the garbage collector must later clean up, and it involves a lot of data copying.

**The Inefficient Way:**
```go
// AVOID THIS PATTERN IN LOOPS
var s string
words := []string{"hello", "world", "this", "is", "inefficient"}
for _, word := range words {
    s += word + " " // Creates a new string in every iteration!
}
```

#### The Solution: `strings.Builder`

A `strings.Builder` is an object designed specifically for this task. It works by maintaining an internal, **mutable** byte slice (`[]byte`).

*   When you write to a `Builder`, you are just appending to this internal slice.
*   The `Builder` intelligently manages the capacity of this slice, often doubling it when it runs out of space. This drastically reduces the number of memory allocations needed.
*   No intermediate string objects are created during the building process.

**The Efficient Way:**
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
    var builder strings.Builder
    words := []string{"hello", "world", "this", "is", "efficient"}

    // You can estimate the final size to pre-allocate the internal slice for max performance.
    // builder.Grow(50) 

    for _, word := range words {
        builder.WriteString(word) // Append the string
        builder.WriteRune(' ')    // Append a single character (rune)
    }

    // Get the final, consolidated string once at the very end.
    // This is the only new string allocation.
    finalString := builder.String()

    fmt.Println(finalString) // Output: hello world this is efficient 
}
```

**Key `strings.Builder` Methods:**

*   `WriteString(s string)`: Appends the contents of a string.
*   `WriteByte(b byte)`: Appends a single byte.
*   `WriteRune(r rune)`: Appends a single rune (and correctly UTF-8 encodes it).
*   `Len()`: Returns the number of bytes currently accumulated.
*   `String()`: Returns the final, accumulated string.

**When to use `strings.Builder`:** Use it any time you are building a string from multiple pieces, especially inside a loop. It is always more performant than using `+=` for string concatenation in such scenarios.