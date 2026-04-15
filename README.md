# Stacks and Queues — Presentation

An interactive slide deck covering stacks, queues, deques, priority queues, monotonic structures, expression evaluation, and concurrent queue designs. Aimed at mid-level software engineers.

## [Open Presentation](https://brendanjameslynskey.github.io/Stacks_and_Queues/index.html)

## [Markdown Version](presentation.md)

---

## Contents

| # | Topic |
|---|-------|
| 01 | Abstract data types vs concrete implementations |
| 02 | The stack — LIFO, push, pop, peek |
| 03 | Array-backed vs linked stack |
| 04 | Stack application — the call stack |
| 05 | Stack application — parentheses matching |
| 06 | Stack application — expression evaluation |
| 07 | Infix to postfix — shunting-yard algorithm |
| 08 | The queue — FIFO, enqueue, dequeue |
| 09 | Array-backed circular buffer |
| 10 | Double-ended queue (deque) |
| 11 | Priority queue |
| 12 | Binary heap implementation |
| 13 | Monotonic stack |
| 14 | Monotonic queue |
| 15 | Min-stack / max-stack in O(1) |
| 16 | Queue using two stacks |
| 17 | Stack using two queues |
| 18 | Concurrent queues — lock-free, CAS |
| 19 | Applications — BFS, scheduling, undo systems |
| 20 | Summary and further reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | append `?print-pdf` to URL |

---

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) (Monokai) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm.

---

## References

- Cormen, T. et al. *Introduction to Algorithms* (CLRS), 4th ed. MIT Press, 2022
- Okasaki, C. *Purely Functional Data Structures*. Cambridge University Press, 1998
- Sedgewick, R. & Wayne, K. *Algorithms*, 4th ed. Addison-Wesley, 2011
- Michael, M. & Scott, M. "Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms." PODC, 1996
- Skiena, S. *The Algorithm Design Manual*, 3rd ed. Springer, 2020

## License

Educational use. Code examples provided as-is.
