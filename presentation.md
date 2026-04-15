# Stacks and Queues

**Computer Science Fundamentals Series**

LIFO | FIFO | Priority queues | Monotonic structures | Deque | Expression evaluation

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [Abstract Data Types vs Concrete Implementations](#slide-02--abstract-data-types-vs-concrete-implementations)
2. [The Stack -- LIFO](#slide-03--the-stack--lifo)
3. [Array-Backed vs Linked Stack](#slide-04--array-backed-vs-linked-stack)
4. [Stack Application -- The Call Stack](#slide-05--stack-application--the-call-stack)
5. [Stack Application -- Parentheses Matching](#slide-06--stack-application--parentheses-matching)
6. [Stack Application -- Expression Evaluation](#slide-07--stack-application--expression-evaluation)
7. [Infix to Postfix -- Shunting-Yard Algorithm](#slide-08--infix-to-postfix--shunting-yard-algorithm)
8. [The Queue -- FIFO](#slide-09--the-queue--fifo)
9. [Array-Backed Circular Buffer](#slide-10--array-backed-circular-buffer)
10. [Double-Ended Queue (Deque)](#slide-11--double-ended-queue-deque)
11. [Priority Queue](#slide-12--priority-queue)
12. [Binary Heap Implementation](#slide-13--binary-heap-implementation)
13. [Monotonic Stack](#slide-14--monotonic-stack)
14. [Monotonic Queue](#slide-15--monotonic-queue)
15. [Min-Stack / Max-Stack in O(1)](#slide-16--min-stack--max-stack-in-o1)
16. [Queue Using Two Stacks](#slide-17--queue-using-two-stacks)
17. [Stack Using Two Queues](#slide-18--stack-using-two-queues)
18. [Concurrent Queues](#slide-19--concurrent-queues)
19. [Applications -- BFS, Scheduling, Undo](#slide-20--applications--bfs-scheduling-undo)
20. [Summary & Further Reading](#slide-21--summary--further-reading)

---

## Slide 02 -- Abstract Data Types vs Concrete Implementations

### Abstract Data Type (ADT)

A mathematical model that defines a data type purely by its **behaviour** -- the operations it supports and their semantics -- without specifying how those operations are implemented.

- Specifies **what** operations exist and what they return
- Says nothing about memory layout, pointers, or arrays
- Language-independent -- a contract, not code

### Concrete implementation

The actual data structure that fulfils an ADT's contract using specific memory layouts.

| ADT | Possible implementations |
|-----|------------------------|
| **Stack** | Dynamic array, singly linked list |
| **Queue** | Circular array buffer, doubly linked list |
| **Priority Queue** | Binary heap, Fibonacci heap, sorted array |
| **Deque** | Circular array buffer, doubly linked list |

> **Key distinction:** the ADT is the *interface*; the data structure is the *implementation*. A stack is not "an array" -- it is an abstraction that can be *backed by* an array.

---

## Slide 03 -- The Stack -- LIFO

Last In, First Out. The most recently added element is the first one removed.

### Core operations

| Operation | Description | Time |
|-----------|------------|------|
| `push(x)` | Place element `x` on top of the stack | `O(1)` |
| `pop()` | Remove and return the top element | `O(1)` |
| `peek()` / `top()` | Return the top element without removing it | `O(1)` |
| `isEmpty()` | Return true if the stack has no elements | `O(1)` |
| `size()` | Return the number of elements | `O(1)` |

### Mental model

```
push(A)  push(B)  push(C)  pop()    peek()
                   ┌───┐
         ┌───┐    │ C │   ┌───┐    ┌───┐
┌───┐   │ B │    │ B │   │ B │    │ B │  ← top
│ A │   │ A │    │ A │   │ A │    │ A │
└───┘   └───┘    └───┘   └───┘    └───┘
                          returns C  returns B
```

---

## Slide 04 -- Array-Backed vs Linked Stack

### Array-backed stack

A dynamic array with a `top` index. Push increments `top` and writes; pop reads and decrements.

- **Pros:** cache-friendly (contiguous memory), zero pointer overhead, amortised `O(1)` push with doubling strategy
- **Cons:** occasional `O(n)` resize when capacity exhausted; wastes space if stack shrinks far below capacity

```python
class ArrayStack:
    def __init__(self):
        self._data = []
    def push(self, x):
        self._data.append(x)
    def pop(self):
        return self._data.pop()
    def peek(self):
        return self._data[-1]
```

### Linked stack

Each node holds a value and a pointer to the node below. Push and pop operate on the head.

- **Pros:** `O(1)` worst-case push/pop (no resize), memory proportional to actual size
- **Cons:** pointer overhead per node, poor cache locality, heap allocation per push

> **In practice:** array-backed stacks win for almost all workloads. The cache advantage dominates. Use a linked stack only when worst-case `O(1)` per-operation matters (real-time systems).

---

## Slide 05 -- Stack Application -- The Call Stack

Every running thread maintains a **call stack** -- a LIFO structure that tracks function invocations.

### What lives in a stack frame

- Return address -- where to resume after the function returns
- Local variables -- primitive values, object references
- Function arguments -- passed by caller
- Saved registers -- CPU state to restore on return

### Stack overflow

Unbounded recursion pushes frames without popping. When the call stack exceeds its allocated memory (typically 1--8 MB), the OS raises a **stack overflow** error.

```python
def factorial(n):
    if n <= 1:         # base case -- recursion stops
        return 1
    return n * factorial(n - 1)   # each call pushes a frame
```

> **Tail-call optimisation** (TCO): some languages/compilers reuse the current frame for a recursive call in tail position, converting recursion to iteration with `O(1)` stack space. Supported in Scheme, Kotlin (`tailrec`), and some C compilers. Not supported in Python or Java.

---

## Slide 06 -- Stack Application -- Parentheses Matching

A classic use: verify that every opening bracket has a correctly nested closing partner.

### Algorithm

```
For each character c in the input:
  if c is '(' or '[' or '{':
      push(c)
  if c is ')' or ']' or '}':
      if stack is empty:
          return INVALID
      if top of stack does not match c:
          return INVALID
      pop()
After all characters:
  return VALID if stack is empty, else INVALID
```

### Trace

```
Input: {[()]}

Step   Char   Stack       Action
1      {      {           push
2      [      {[          push
3      (      {[(         push
4      )      {[          pop -- matches (
5      ]      {           pop -- matches [
6      }      (empty)     pop -- matches {
→ VALID
```

> Generalises to any nested delimiter: HTML tags, XML, begin/end blocks. The stack enforces the constraint that closers must match the most recent opener.

---

## Slide 07 -- Stack Application -- Expression Evaluation

### Postfix (Reverse Polish Notation) evaluation

Postfix eliminates the need for parentheses and operator precedence rules. Evaluate left to right using a stack.

```
Expression:  3 4 + 2 * 7 -

Step   Token   Stack          Action
1      3       [3]            push operand
2      4       [3, 4]         push operand
3      +       [7]            pop 4 and 3, push 3+4=7
4      2       [7, 2]         push operand
5      *       [14]           pop 2 and 7, push 7*2=14
6      7       [14, 7]        push operand
7      -       [7]            pop 7 and 14, push 14-7=7

Result: 7
```

### Algorithm

```
For each token:
  if token is a number:
      push(token)
  if token is an operator:
      b = pop()
      a = pop()
      push(a op b)
Return pop()  -- the final result
```

> Used by HP calculators, Forth, PostScript, and the Java bytecode verifier (which models the operand stack).

---

## Slide 08 -- Infix to Postfix -- Shunting-Yard Algorithm

Dijkstra's shunting-yard algorithm converts infix expressions (e.g. `3 + 4 * 2`) to postfix using a stack.

### Rules

```
For each token:
  NUMBER  → output directly
  (       → push onto operator stack
  )       → pop and output until '(' is found; discard '('
  OPERATOR→ while top of stack has higher or equal precedence:
               pop and output
            push current operator
After all tokens:
  pop and output remaining operators
```

### Worked example

```
Infix:  3 + 4 * 2 - ( 1 + 5 )

Token   Output queue         Operator stack   Action
3       3                                     output
+       3                    +                push op
4       3 4                  +                output
*       3 4                  + *              push (* > +)
2       3 4 2                + *              output
-       3 4 2 * +            -                pop * and +, push -
(       3 4 2 * +            - (              push (
1       3 4 2 * + 1          - (              output
+       3 4 2 * + 1          - ( +            push op
5       3 4 2 * + 1 5        - ( +            output
)       3 4 2 * + 1 5 +      -                pop + until (, discard (
END     3 4 2 * + 1 5 + -                     pop remaining

Postfix: 3 4 2 * + 1 5 + -
```

---

## Slide 09 -- The Queue -- FIFO

First In, First Out. Elements are added at the rear and removed from the front.

### Core operations

| Operation | Description | Time |
|-----------|------------|------|
| `enqueue(x)` | Add element `x` to the rear | `O(1)` |
| `dequeue()` | Remove and return the front element | `O(1)` |
| `front()` / `peek()` | Return the front element without removing it | `O(1)` |
| `isEmpty()` | Return true if the queue has no elements | `O(1)` |
| `size()` | Return the number of elements | `O(1)` |

### Mental model

```
enqueue(A)  enqueue(B)  enqueue(C)  dequeue()
                                     returns A
front                    front        front
  ↓                        ↓            ↓
┌───┐     ┌───┬───┐     ┌───┬───┬───┐  ┌───┬───┐
│ A │     │ A │ B │     │ A │ B │ C │  │ B │ C │
└───┘     └───┴───┘     └───┴───┴───┘  └───┴───┘
                                rear        rear
```

> The queue is the natural structure for fairness -- first come, first served. Every message broker, task runner, and print spooler is built on this principle.

---

## Slide 10 -- Array-Backed Circular Buffer

A fixed-size array with `head` and `tail` pointers that wrap around, avoiding the `O(n)` cost of shifting elements.

### How it works

```
Capacity: 5     head=0, tail=0  (empty)

enqueue(A): [A _ _ _ _]   head=0, tail=1
enqueue(B): [A B _ _ _]   head=0, tail=2
enqueue(C): [A B C _ _]   head=0, tail=3
dequeue():  [_ B C _ _]   head=1, tail=3   returns A
enqueue(D): [_ B C D _]   head=1, tail=4
enqueue(E): [_ B C D E]   head=1, tail=0   ← wraps around
enqueue(F): [F B C D E]   head=1, tail=1   ← wraps around
```

### Key formulas

```
tail = (tail + 1) % capacity      // advance tail on enqueue
head = (head + 1) % capacity      // advance head on dequeue
size = (tail - head + capacity) % capacity
full = (size == capacity - 1)     // leave one slot empty to distinguish full from empty
```

> **Why circular?** A naive array queue wastes all space before the head pointer. Shifting elements on dequeue is `O(n)`. The circular buffer solves both problems with modular arithmetic.

---

## Slide 11 -- Double-Ended Queue (Deque)

A deque (pronounced "deck") supports insertion and removal at **both** ends in `O(1)`.

### Operations

| Operation | Description |
|-----------|------------|
| `pushFront(x)` | Insert at the front |
| `pushBack(x)` | Insert at the rear |
| `popFront()` | Remove from the front |
| `popBack()` | Remove from the rear |
| `peekFront()` | View front element |
| `peekBack()` | View rear element |

### Implementations

- **Circular array buffer** -- same as queue, but head can move backwards too
- **Doubly linked list** -- natural fit, insert/remove at either end is `O(1)`
- **Block-based** -- `std::deque` in C++ uses a map of fixed-size blocks for pointer stability and cache friendliness

### Deque subsumes stack and queue

A deque restricted to one end is a stack. A deque restricted to pushBack + popFront is a queue. This is why Python's `collections.deque` is recommended for both use cases.

> Java: `ArrayDeque` (preferred) or `LinkedList`. C++: `std::deque`. Python: `collections.deque`. All outperform a plain list for queue operations.

---

## Slide 12 -- Priority Queue

A collection where each element has a **priority** and the highest-priority element is always dequeued first.

### Operations

| Operation | Description | Heap time |
|-----------|------------|-----------|
| `insert(x, p)` | Add element with priority | `O(log n)` |
| `extractMax()` / `extractMin()` | Remove highest-priority element | `O(log n)` |
| `peek()` | View highest-priority element | `O(1)` |
| `changePriority(x, p')` | Update an element's priority | `O(log n)` |

### Not a sorted list

A sorted array gives `O(1)` extract but `O(n)` insert. A heap gives `O(log n)` for both -- the right trade-off for dynamic workloads.

### Use cases

- **Dijkstra's shortest path** -- extract the nearest unvisited vertex
- **A* search** -- extract the node with lowest `f(n) = g(n) + h(n)`
- **Huffman coding** -- repeatedly extract two lowest-frequency nodes
- **OS scheduling** -- extract the highest-priority runnable thread
- **Event-driven simulation** -- extract the event with the earliest timestamp

---

## Slide 13 -- Binary Heap Implementation

A binary heap is a **complete binary tree** stored in a flat array. No pointers needed.

### Array layout

```
Index:   0   1   2   3   4   5   6
Value: [10, 15, 20, 25, 30, 35, 40]   (min-heap)

          10            Parent of i:  (i-1) / 2
         /  \           Left child:   2i + 1
       15    20         Right child:  2i + 2
      / \    / \
    25  30  35  40
```

### Heap operations

**Sift-up** (after insert): place new element at end, swap with parent while smaller (min-heap). `O(log n)`.

**Sift-down** (after extract): move last element to root, swap with smallest child while larger. `O(log n)`.

**Heapify** (build heap from array): call sift-down on each non-leaf node from bottom up. `O(n)` -- not `O(n log n)`.

### Why O(n) heapify?

Most nodes are near the bottom and sift down only 1--2 levels. The sum of work across all levels forms a convergent geometric series:

```
n/4 * 1 + n/8 * 2 + n/16 * 3 + ... = O(n)
```

---

## Slide 14 -- Monotonic Stack

A stack that maintains elements in **monotonically increasing or decreasing** order. When a new element violates the monotonicity, elements are popped until order is restored.

### Pattern: Next Greater Element

For each element in an array, find the first element to its right that is greater.

```python
def next_greater(nums):
    n = len(nums)
    result = [-1] * n
    stack = []            # stores indices, values are decreasing
    for i in range(n):
        while stack and nums[i] > nums[stack[-1]]:
            idx = stack.pop()
            result[idx] = nums[i]
        stack.append(i)
    return result

# nums   = [2, 1, 4, 3, 5]
# result = [4, 4, 5, 5, -1]
```

### Why it works

Each element is pushed once and popped at most once. Total work: `O(n)` despite the nested loop.

### Classic problems

- **Next greater / smaller element** (left or right)
- **Largest rectangle in histogram** -- `O(n)` using a monotonic increasing stack
- **Trapping rain water** -- can be solved with two monotonic stacks or two pointers
- **Daily temperatures** -- days until a warmer day

---

## Slide 15 -- Monotonic Queue

A deque that maintains elements in monotonic order, enabling **sliding-window min/max** queries in `O(n)` total.

### Sliding window maximum

```python
from collections import deque

def max_sliding_window(nums, k):
    dq = deque()     # stores indices; values are decreasing
    result = []
    for i, x in enumerate(nums):
        # remove elements outside the window
        while dq and dq[0] < i - k + 1:
            dq.popleft()
        # maintain decreasing order
        while dq and nums[dq[-1]] <= x:
            dq.pop()
        dq.append(i)
        if i >= k - 1:
            result.append(nums[dq[0]])
    return result

# nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3
# result = [3, 3, 5, 5, 6, 7]
```

### Complexity

Each element enters and leaves the deque at most once. Total: `O(n)` time, `O(k)` space.

> A naive approach checks all `k` elements per window position: `O(nk)`. The monotonic deque eliminates redundant comparisons by discarding elements that can never be the maximum.

---

## Slide 16 -- Min-Stack / Max-Stack in O(1)

Track the minimum (or maximum) element at every point in time using an auxiliary stack.

### Approach 1: Auxiliary stack

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []    # parallel stack of current minimums

    def push(self, x):
        self.stack.append(x)
        m = min(x, self.min_stack[-1]) if self.min_stack else x
        self.min_stack.append(m)

    def pop(self):
        self.min_stack.pop()
        return self.stack.pop()

    def getMin(self):
        return self.min_stack[-1]     # O(1)
```

### Approach 2: Single stack with encoded values

Store `2 * x - current_min` when `x < current_min`. On pop, if the stored value is less than the current min, decode the previous min. Uses `O(1)` extra space but risks integer overflow.

### All operations: `O(1)` time

| Operation | Main stack | Min stack |
|-----------|-----------|-----------|
| `push(x)` | push `x` | push `min(x, current_min)` |
| `pop()` | pop | pop |
| `getMin()` | -- | peek |

---

## Slide 17 -- Queue Using Two Stacks

Implement a FIFO queue using two LIFO stacks. Amortised `O(1)` per operation.

### Strategy

- **Stack IN** -- receives all enqueued elements
- **Stack OUT** -- serves all dequeue requests
- When OUT is empty, pour all of IN into OUT (reversing the order)

### Trace

```
enqueue(1):  IN=[1]         OUT=[]
enqueue(2):  IN=[1,2]       OUT=[]
dequeue():   IN=[]          OUT=[2,1]  ← pour
             IN=[]          OUT=[2]    → returns 1
enqueue(3):  IN=[3]         OUT=[2]
dequeue():   IN=[3]         OUT=[]     → returns 2
dequeue():   IN=[]          OUT=[3]    ← pour
             IN=[]          OUT=[]     → returns 3
```

### Amortised analysis

Each element is moved from IN to OUT exactly once. Over `n` operations, total moves = `n`. Therefore each operation is **amortised `O(1)`**.

> This is a popular interview question, but it also has real uses: functional languages without mutable arrays implement queues this way (e.g. Okasaki's persistent queue).

---

## Slide 18 -- Stack Using Two Queues

Implement a LIFO stack using two FIFO queues. Two strategies exist.

### Strategy 1: Costly push

Make push `O(n)` and pop `O(1)`.

```
push(x):
    enqueue x into Q2
    while Q1 is not empty:
        dequeue from Q1, enqueue into Q2
    swap Q1 and Q2

pop():
    dequeue from Q1
```

### Strategy 2: Costly pop

Make push `O(1)` and pop `O(n)`.

```
push(x):
    enqueue x into Q1

pop():
    while Q1 has more than 1 element:
        dequeue from Q1, enqueue into Q2
    result = dequeue from Q1
    swap Q1 and Q2
    return result
```

### Comparison

| Strategy | push | pop | Best when |
|----------|------|-----|-----------|
| Costly push | `O(n)` | `O(1)` | Many more pops than pushes |
| Costly pop | `O(1)` | `O(n)` | Many more pushes than pops |

> Neither approach achieves amortised `O(1)` for both operations. This is why the two-stack queue is preferred in practice -- it achieves amortised `O(1)` for all operations.

---

## Slide 19 -- Concurrent Queues

In multi-threaded systems, queues must handle simultaneous producers and consumers safely.

### Lock-based queue

Wrap enqueue/dequeue in a mutex. Simple but creates a bottleneck -- all threads contend on one lock.

### Lock-free queue (Michael & Scott, 1996)

Uses **Compare-And-Swap (CAS)** instructions to atomically update head and tail pointers without locks.

```
CAS(addr, expected, new):
    atomically:
        if *addr == expected:
            *addr = new
            return true
        return false
```

### Key idea

- **Enqueue:** CAS the tail's `next` pointer from null to the new node, then CAS the tail pointer forward
- **Dequeue:** CAS the head pointer from the current head to `head.next`
- If CAS fails, another thread succeeded -- retry (spin)

### Trade-offs

| Approach | Throughput | Fairness | Complexity |
|----------|-----------|----------|-----------|
| Mutex queue | Moderate | FIFO with OS scheduling | Low |
| Lock-free (CAS) | High | Not guaranteed | High |
| Wait-free | Highest (bounded latency) | Guaranteed progress | Very high |

> Java: `ConcurrentLinkedQueue` (lock-free). Go: channels (lock-based, CSP model). Rust: `crossbeam::queue` (lock-free). C++: `boost::lockfree::queue`.

---

## Slide 20 -- Applications -- BFS, Scheduling, Undo

### Breadth-First Search (BFS)

A queue drives BFS: enqueue the start node, then repeatedly dequeue a node, process it, and enqueue its unvisited neighbours. Guarantees shortest path in unweighted graphs.

### Task scheduling

- **Round-robin CPU scheduling** -- processes cycle through a ready queue, each getting a time slice
- **Thread pool work queues** -- tasks enqueued by producers, dequeued by worker threads
- **Message queues** -- Kafka, RabbitMQ, SQS: producers enqueue messages, consumers dequeue them asynchronously

### Undo / Redo systems

A stack-based pattern: every user action is pushed onto the **undo stack**. On undo, the action is popped and pushed onto the **redo stack**. On redo, the reverse.

```
Action → push to undo stack
Undo   → pop undo, push to redo
Redo   → pop redo, push to undo
New action after undo → clear redo stack (branch discarded)
```

### Summary of structure-to-application mapping

| Structure | Application |
|-----------|------------|
| Stack | Call stack, undo, expression eval, DFS, backtracking |
| Queue | BFS, scheduling, buffering, message passing |
| Priority queue | Dijkstra, A*, Huffman, OS scheduling |
| Deque | Sliding window, work stealing, palindrome checking |

---

## Slide 21 -- Summary & Further Reading

### Key takeaways

- Stacks and queues are ADTs -- define them by operations and invariants, not by implementation
- Array-backed implementations dominate in practice due to cache locality
- The circular buffer solves the dequeue-shift problem with modular arithmetic
- Monotonic stacks and queues reduce common array problems from `O(n^2)` to `O(n)`
- A min-stack augments the stack ADT with `O(1)` minimum queries using a parallel tracking stack
- Two stacks make a queue with amortised `O(1)` -- a foundational result in data structure design
- Priority queues (binary heaps) are the engine behind shortest-path and greedy algorithms
- Lock-free concurrent queues use CAS to avoid mutex bottlenecks in high-throughput systems

### Recommended reading

| Source | Description |
|--------|------------|
| **Cormen et al.** | *Introduction to Algorithms* (CLRS) -- chapters 6 (heaps), 10 (stacks/queues), 20 (Fibonacci heaps) |
| **Okasaki** | *Purely Functional Data Structures* -- persistent stacks, queues, and amortised analysis |
| **Sedgewick** | *Algorithms*, 4th ed. -- practical Java implementations of all structures covered here |
| **Michael & Scott** | "Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms" (1996) |
| **Skiena** | *The Algorithm Design Manual* -- when to use which structure, with war stories |
