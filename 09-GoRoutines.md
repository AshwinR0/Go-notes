### 1. Concurrency vs. Parallelism

These two terms are often used interchangeably, but they represent different concepts.

*   **Concurrency:** Concurrency is about **dealing with** lots of things at once. It's a way of structuring your program to handle multiple tasks in overlapping time periods. Think of a barista at a coffee shop who starts one customer's espresso, then takes the next customer's order while the espresso machine is running, then froths milk for the first customer. They are *managing* multiple tasks, but may only be *doing* one thing at any given microsecond. Concurrency is about the *structure* of the program.

*   **Parallelism:** Parallelism is about **doing** lots of things at once. It requires hardware with multiple processing units (e.g., a multi-core CPU). Think of two baristas working side-by-side, each making a coffee for a different customer at the exact same time. Parallelism is about the *execution* of the program.

**Go's Approach:** Go makes it easy to write **concurrent** code using goroutines. The Go runtime scheduler then takes these concurrent tasks and runs them in **parallel** on available CPU cores, giving you the best of both worlds.

---

### 2. Goroutines

A **goroutine** is an incredibly lightweight thread of execution managed by the Go runtime, not the operating system.

*   **Lightweight:** A goroutine starts with only a few kilobytes of stack space, which can grow and shrink as needed. This is much smaller than a traditional OS thread. It's common and efficient to have hundreds of thousands of goroutines running in a single program.
*   **Easy to Create:** You can start a new goroutine by simply using the `go` keyword before a function call.

```go
package main

import (
	"fmt"
	"time"
)

func sayHello() {
	fmt.Println("Hello from the goroutine!")
}

func main() {
	// Start a new goroutine to run the sayHello function.
	go sayHello()

	fmt.Println("Hello from the main function!")

	// PROBLEM: The main function might exit before the goroutine has a chance to run.
	// We add a sleep here to demonstrate, but this is a BAD way to synchronize.
	time.Sleep(1 * time.Second) 
}```
This example highlights a key problem: how do we wait for our goroutines to finish? That's where `WaitGroup` comes in.

---

### 3. `sync.WaitGroup`

A `WaitGroup` is a simple but powerful tool from the `sync` package that allows you to wait for a collection of goroutines to finish executing. It works like a counter.

*   `wg.Add(n)`: Increases the WaitGroup counter by `n`. You call this *before* starting your goroutines to tell the WaitGroup how many tasks to wait for.
*   `wg.Done()`: Decrements the WaitGroup counter by one. You typically call this at the end of each goroutine using `defer` to signal that it has completed its work.
*   `wg.Wait()`: Blocks the execution of the program until the WaitGroup counter becomes zero. This is usually called in the `main` function to wait for all background work to be done.

**Corrected Example with WaitGroup:**
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// The worker function now accepts a WaitGroup pointer.
func worker(id int, wg *sync.WaitGroup) {
	// Use defer to ensure Done() is called when the function exits.
	defer wg.Done()

	fmt.Printf("Worker %d starting\n", id)
	time.Sleep(time.Second) // Simulate some work
	fmt.Printf("Worker %d finished\n", id)
}

func main() {
	var wg sync.WaitGroup // Create a WaitGroup

	for i := 1; i <= 3; i++ {
		wg.Add(1) // Increment the counter for each goroutine we're about to start.
		go worker(i, &wg)
	}

	fmt.Println("Main: Waiting for workers to finish...")
	wg.Wait() // Block here until the counter is 0.
	fmt.Println("Main: All workers have finished.")
}
```

---

### 4. `sync.Mutex` and Why It's Required

When multiple goroutines access and modify the same shared data (a variable, a struct, a map, etc.) at the same time, you can get a **race condition**. This leads to unpredictable and incorrect results because the order of operations is not guaranteed.

**Why is a Mutex Required?** A mutex (short for **mut**ual **ex**clusion) is a lock that ensures only one goroutine can access a specific piece of code at a time. This protected section of code is called a **critical section**.

**The Problem (Race Condition):**
```go
var counter = 0
// If 100 goroutines call counter++ concurrently, the final result will
// almost certainly NOT be 100. This is because counter++ is not an
// atomic operation. It is actually three steps:
// 1. Read the current value of 'counter'.
// 2. Add 1 to that value.
// 3. Write the new value back to 'counter'.
// Two goroutines could both read the value '5', both increment it to '6',
// and both write back '6'. You've lost an increment.
```

#### `m.Lock()` and `m.Unlock()`

*   `m.Lock()`: Acquires the lock. If another goroutine already holds the lock, this call will block until the lock is released.
*   `m.Unlock()`: Releases the lock, allowing other waiting goroutines to acquire it.

**Corrected Example with Mutex:**
```go
package main

import (
	"fmt"
	"sync"
)

var counter = 0

func main() {
	var wg sync.WaitGroup
	var m sync.Mutex // Create a Mutex

	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			
			m.Lock() // Lock before accessing the shared 'counter'
			counter++
			m.Unlock() // Unlock after accessing it
		}()
	}

	wg.Wait()
	fmt.Println("Final counter value:", counter) // Output will now be 1000
}```

#### Positioning of Lock and Unlock (Best Practice: `defer`)

It is critical to unlock a mutex after you are done with the critical section. If you forget, your program will suffer a **deadlock**—all other goroutines waiting for the lock will wait forever.

The safest and most idiomatic way to handle this in Go is to use `defer`. A deferred function call is executed just before the surrounding function returns.

**Correct Positioning and `defer`:**
```go
func increment() {
    m.Lock()
    defer m.Unlock() // THIS IS THE BEST PRACTICE!

    // The Unlock() call is now guaranteed to happen when this function
    // returns, even if there are multiple return paths or a panic.

    counter++
    // No need to call m.Unlock() here. Defer handles it.
}
```

### 5. `sync.RWMutex` (Read-Write Mutex)

A standard `Mutex` is exclusive: only one goroutine can hold the lock, period. This can be inefficient if you have a piece of data that is **read very often** but **written to only rarely**.

An `RWMutex` (Read-Write Mutex) provides a more granular lock:
*   It allows any number of **readers** to access the data at the same time (`RLock`).
*   It allows only one **writer** at a time, and it blocks all readers while the write lock is held (`Lock`).

This is perfect for things like a configuration cache that many goroutines need to read from, but which is only updated occasionally.

*   `m.RLock()`: Acquires a read lock. Multiple goroutines can hold a read lock simultaneously.
*   `m.RUnlock()`: Releases a read lock.
*   `m.Lock()`: Acquires an exclusive write lock.
*   `m.Unlock()`: Releases a write lock.

**Example Use Case:**
```go
var config map[string]string
var configLock sync.RWMutex

// Many goroutines can call this at the same time.
func readConfig(key string) string {
    configLock.RLock()
    defer configLock.RUnlock()
    return config[key]
}

// Only one goroutine at a time can call this.
// All readers will be blocked while this is running.
func writeConfig(key, value string) {
    configLock.Lock()
    defer configLock.Unlock()
    config[key] = value
}
```