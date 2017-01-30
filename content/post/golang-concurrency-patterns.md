+++
date = "2015-12-01T21:45:02-08:00"
title = "Golang Concurrency Pattern"

+++

## A pattern for jobs that need a worker pool.

Rule 1 of good concurrency in Go: thou shalt use channels and don’t even think 
about sending a pointer down it!

OK, so being totally rigid about this rule is basically impossible (but 
seriously, don’t sent pointers down channels, that bit is set in stone) and 
will actually increase the complexity of your code for those tasks where a 
simple mutex suffices, especially when Go’s defer enables you to write the lock
and unlock on consecutive lines, making it easy to see if you omitted something 
(you could even write a simple tool to check it for you as part of your CI).

When the conversation moves to worker pools and the like, channels are the way 
to go every time. Channels are tightly integrated into a lot of Go’s core 
concepts, almost to the point that some things one doesn’t expect to work turn 
out to work exactly as desired. 

There are three main ideas that have to be addressed in the pursuit of creating
effective worker pools with simple code:

  1. How do we get work to the workers?
  2. How do we signal the workers to shut down?
  3. How do we ensure the application shuts down only after all the workers have completed?

The solution to 1. should be obvious from the opening: we’re going to use 
channels! Questions 2. and 3. will be addressed by some of that tight 
integration of channels, and WaitGroups respectively. I’ve intentionally 
left out any concept of how workers signal for more work, instead favouring 
an approach where the channel is always primed with a pending task (when there 
is one available). Back channels can be added without too much complication to 
allow workers to signal back if it is required.

Let’s have a look at the code for a simple worker pool setup:

```
package main

import (
	"fmt"
	"math/rand"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

const workers = 5

func main() {
	wg := new(sync.WaitGroup)
	// using a capacity of 0 ensures only one job is ever in a pending state
	c := make(chan int, 0)
	s := make(chan os.Signal)
	defer close(s)

	// We'll use SIGHUP as our trigger to shutdown so that <ctrl>-c kills
	// the process
	signal.Notify(s, syscall.SIGHUP)

	for i := 0; i < workers; i++ {
		go work(wg, i, c)
	}

	sendJobs(c, s)

	// by closing the channel the workers are performing
	// a range over, the for loop in the workers automatically
	// exits
	close(c)

	// wait for all the workers to complete
	wg.Wait()
}

func work(wg *sync.WaitGroup, id int, c <-chan int) {
	// doing both the Add and Done here ensures the symmetry of
	// WaitGroup operations.
	wg.Add(1)
	defer wg.Done()

	// range over a channel!
	for job := range c {
		fmt.Printf("Worker %d received job: %d\n", id, job)
		time.Sleep(time.Duration(rand.Int()%5) * time.Second)
	}
}

func sendJobs(c chan<- int, s <-chan os.Signal) {
	for i := 0; ; i++ {
		select {
		case <-s:
			// if we read anything on s, it means we received a SIGINT
			// and we want to stop sending work.
			return
		default:
			// if there's nothing on s, we're going to publish more work.
			c <- i
		}
	}
}
```

This basic structure can be applied to most worker pool type tasks. It may not 
be an os signal that triggers your shutdown, but the pattern remains the same. 
The most interesting bit in my opinion is the synergy of `for` with `range myChannel` 
and `close(myChannel)`. This alone is an incredibly powerful structure
for managing the lifetime of channels.