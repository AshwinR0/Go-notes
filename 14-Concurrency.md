### 1. The For-Select Loop: The Concurrent Worker's Engine

The `for-select` loop is the most common and idiomatic way to create a long-running goroutine (a "worker" or "daemon") that needs to respond to multiple events from different channels.

**What it is:** It's an infinite `for` loop that contains a single `select` statement.

**Purpose:** It allows a goroutine to wait on multiple channel operations simultaneously. The goroutine blocks until one of the cases in the `select` statement can proceed, processes that event, and then loops back to wait for the next event.

**Analogy:** Think of a factory worker at a workstation. They are in a perpetual loop of "wait for something to do." They can either:
*   Receive a new part from a conveyor belt (`case part := <-partsChannel`).
*   Receive a stop signal from their supervisor (`case <-stopSignalChannel`).

The worker just waits, and whichever event happens first, they react to it.

**Code Structure:**
```go
func worker(jobs <-chan int, results chan<- int) {
	for { // Loop forever
		select {
		case job := <-jobs:
			// A job was received. Process it.
			result := process(job)
			results <- result // Send the result back.
		// ... more cases for other channels could go here ...
		}
	}
}
```

**The Problem with this simple loop:** It runs forever. There is no way to tell the `worker` goroutine to stop cleanly. If the `main` goroutine exits, this worker is abruptly terminated. This leads us to the next pattern.

---

### 2. The Done Channel Pattern: The Braking System

The **done channel** is the standard, idiomatic Go pattern for signaling a goroutine to gracefully shut down.

**What it is:** A channel, typically of type `chan struct{}`, that is passed into a goroutine. The owner of the goroutine closes this channel to signal that the goroutine should stop its work, clean up its resources, and exit.

**Mechanism:**

1.  **Ownership:** The goroutine that creates the worker (the "owner") also creates the `done` channel.
2.  **Signaling:** The worker uses a `case <-done:` in its `for-select` loop.
3.  **Shutdown:** When the owner wants the worker to stop, it simply calls `close(done)`.
4.  **Broadcast:** A receive on a closed channel **always succeeds immediately** and returns a zero value. This makes `close()` a perfect broadcast mechanism—a single `close` will unblock *every* goroutine that is listening on that `done` channel.

**Analogy:** The `done` channel is like a factory-wide fire alarm. The factory manager (`main` goroutine) pulls the alarm (`close(done)`). Every worker waiting for the alarm (`case <-done`) immediately stops what they're doing, cleans up their station, and exits the building (`return`).

**Code Example with Done Channel:**
```go
package main

import (
	"fmt"
	"time"
)

func worker(done <-chan struct{}, jobs <-chan int) {
	for {
		select {
		case job := <-jobs:
			fmt.Println("Processing job:", job)
			time.Sleep(500 * time.Millisecond) // Simulate work
		case <-done:
			// The done channel was closed.
			fmt.Println("Worker is shutting down...")
			return // Exit the loop and the goroutine.
		}
	}
}

func main() {
	done := make(chan struct{})
	jobs := make(chan int)

	// Start the worker
	go worker(done, jobs)

	// Send a few jobs
	for i := 1; i <= 3; i++ {
		jobs <- i
		fmt.Println("Sent job:", i)
	}

	// Wait a moment, then signal the worker to stop.
	time.Sleep(2 * time.Second)
	fmt.Println("Signaling worker to stop...")
	close(done) // This triggers the `case <-done` in the worker.

	// Give the worker a moment to print its shutdown message
	time.Sleep(500 * time.Millisecond)
	fmt.Println("Main function finished.")
}
```

---

### 3. Pipelines: The Assembly Line

A **pipeline** is a design pattern where a series of stages (our worker goroutines) are connected by channels. The output of one stage becomes the input for the next. It's a highly effective way to structure concurrent data processing.

**Benefits:**
*   **Concurrency:** Each stage runs in its own goroutine, processing data in parallel.
*   **Modularity:** Each stage is a self-contained unit of work. It only needs to know about its input and output channels.
*   **Backpressure:** If one stage is slow, the channels connecting it will fill up (or block if unbuffered), naturally slowing down the upstream stages. This prevents the system from being overwhelmed.

**Analogy:** A car factory assembly line.
*   **Stage 1 (Generator):** Puts raw steel chassis onto the first conveyor belt.
*   **Stage 2 (Processor):** Takes a chassis from the first belt, adds an engine, and puts the result onto a second belt.
*   **Stage 3 (Processor):** Takes a chassis+engine from the second belt, adds wheels, and puts it on a final belt.
*   **The `done` channel** is the emergency stop button for the entire assembly line.

**Putting It All Together: A Pipeline Example**

Let's build a simple pipeline that:
1.  **Stage 1 (`generator`):** Produces numbers.
2.  **Stage 2 (`squarer`):** Receives numbers and sends their squares.
3.  **Main (`sink`):** Receives the squared numbers and prints them.

```go
package main

import (
	"fmt"
	"sync"
)

// Stage 1: Generates numbers and sends them to the 'out' channel.
// It stops when the 'done' channel is closed.
func generator(done <-chan struct{}, nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out) // Close the out channel when the goroutine exits.
		for _, n := range nums {
			select {
			case out <- n:
			case <-done:
				return
			}
		}
	}()
	return out
}

// Stage 2: Receives numbers from the 'in' channel, squares them,
// and sends them to the 'out' channel.
func squarer(done <-chan struct{}, in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			select {
			case out <- n * n:
			case <-done:
				return
			}
		}
	}()
	return out
}

func main() {
	done := make(chan struct{})
	defer close(done) // Ensure we signal shutdown on exit.

	// Set up the pipeline.
	in := generator(done, 2, 3, 4, 5)
	out := squarer(done, in)

	// Consume the output from the final stage.
	// This is the "sink".
	for n := range out {
		fmt.Println(n) // Prints 4, 9, 16, 25
	}
}
```In this final example, the `done` channel provides a way to tear down the entire pipeline gracefully. If `main` closed the `done` channel early, the signal would propagate through all stages, causing them to stop processing and exit cleanly.