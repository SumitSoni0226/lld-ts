# Low-Level Design in TypeScript

My journey of learning **Low-Level Design (LLD) from scratch using TypeScript**, with the goal of developing strong object-oriented and software design skills for **SDE2-level interviews**.

The focus is not on memorizing design patterns.

The focus is on learning **how to think about software design**.

---

## 🎯 Goal

Develop the ability to take a problem with changing requirements and:

* Identify the right objects/classes
* Define responsibilities clearly
* Decide how objects should interact
* Write maintainable and extensible TypeScript
* Recognize bad design and improve it
* Understand when and why to use design principles
* Understand when a design pattern actually solves a problem
* Explain design decisions clearly in an interview

The ultimate goal is to be able to approach an LLD problem like:

```text
Requirements
     ↓
Understand the problem
     ↓
Identify entities
     ↓
Define responsibilities
     ↓
Design relationships
     ↓
Write initial design
     ↓
Requirements change
     ↓
Identify what breaks
     ↓
Improve the design
     ↓
Apply principles/patterns when needed
     ↓
Final design
     ↓
TypeScript implementation
```

---

# 🧠 Learning Philosophy

I am learning LLD from scratch.

I don't want to simply memorize:

> "SOLID has 5 principles."

or:

> "Strategy is a design pattern."

Instead, I want to understand **why these concepts exist**.

Therefore, problems will be introduced first.

The learning process will generally be:

```text
Problem
   ↓
My initial approach
   ↓
Implement a simple solution
   ↓
Requirement changes
   ↓
Design starts becoming difficult
   ↓
Understand the underlying problem
   ↓
Learn the relevant principle/pattern
   ↓
Redesign
   ↓
Implement in TypeScript
```

The objective is to develop **design intuition**, not pattern-recitation skills.

---

# 🗺️ Roadmap

## Phase 0 — Understand the Basics

* [ ] Class & Object
* [ ] Interface
* [ ] Encapsulation
* [ ] Abstraction
* [ ] Inheritance
* [ ] Polymorphism
* [ ] Composition
* [ ] Composition vs Inheritance

---

## Phase 1 — Design Thinking

Learn how to go from a problem statement to a basic object-oriented design.

* [ ] Identify entities
* [ ] Identify responsibilities
* [ ] Define relationships
* [ ] Think about object interactions
* [ ] Start with a simple design
* [ ] Handle changing requirements
* [ ] Identify design smells
* [ ] Refactor the design

### Problems

| # | Problem        | Concepts Discovered | Status |
| - | -------------- | ------------------- | ------ |
| 1 | Coffee Machine | —                   | ⬜      |
| 2 | TBD            | —                   | ⬜      |
| 3 | TBD            | —                   | ⬜      |

---

# Phase 2 — SOLID

Learn SOLID principles through problems rather than memorization.

* [ ] Single Responsibility Principle
* [ ] Open/Closed Principle
* [ ] Liskov Substitution Principle
* [ ] Interface Segregation Principle
* [ ] Dependency Inversion Principle

For every principle, understand:

```text
Problem
   ↓
Bad design
   ↓
Why it becomes difficult
   ↓
Principle
   ↓
Improved design
```

---

# Phase 3 — Design Patterns

Learn patterns only when there is a problem that benefits from them.

### Creational

* [ ] Factory
* [ ] Builder
* [ ] Singleton

### Structural

* [ ] Adapter
* [ ] Decorator

### Behavioral

* [ ] Strategy
* [ ] Observer
* [ ] State
* [ ] Command
* [ ] Template Method

For every pattern, understand:

```text
What problem does it solve?
Why does the naive approach become difficult?
What does the pattern change?
When should I NOT use it?
```

---

# Phase 4 — LLD Problems

Apply everything learned to real interview-style problems.

### Beginner

* [ ] Coffee Machine
* [ ] Tic-Tac-Toe
* [ ] Vending Machine

### Intermediate

* [ ] Parking Lot
* [ ] Library Management System
* [ ] Snake & Ladder
* [ ] Elevator System

### Advanced

* [ ] ATM
* [ ] Splitwise
* [ ] Movie Ticket Booking
* [ ] Chess
* [ ] Notification System

The list will evolve as I learn.

---

# 📝 Problem Documentation Format

Each major problem will be documented as a Markdown file.

Example:

```text
04-lld-problems/
└── parking-lot.md
```

Each problem should capture the **evolution of the design**, not just the final solution.

A typical note will contain:

```text
1. Problem
2. Requirements
3. My First Thought
4. Initial Design
5. Initial TypeScript Code
6. Requirement Changes
7. What Broke?
8. Why Did It Break?
9. Design Improvement
10. Principle / Pattern Discovered
11. Final Design
12. Final TypeScript Implementation
13. Design Decisions
14. What I Learned
15. Interview Questions
16. Things I Still Don't Understand
```

---

# 📂 Repository Structure

```text
lld-typescript/
│
├── README.md
│
├── 00-basics/
│
├── 01-design-thinking/
│
├── 02-solid/
│
├── 03-design-patterns/
│
├── 04-lld-problems/
│
└── code/
```

The repository will grow gradually as I learn.

I will not create files for concepts that I haven't learned yet.

---

# 💻 Language

All implementations will be written in:

**TypeScript**

The focus will be on writing clean, object-oriented, maintainable TypeScript rather than just making the code work.

---

# 📚 Learning Notes → PDF

The Markdown files will eventually be converted into learning PDFs.

The PDFs should preserve the learning journey:

```text
Problem
   ↓
Initial thinking
   ↓
Mistake / limitation
   ↓
Requirement change
   ↓
Design improvement
   ↓
Principle / Pattern
   ↓
Final design
   ↓
Code
```

This will allow me to use the notes later for revision before interviews.

---

# 🎯 Interview Goal

By the end of this journey, I should be able to receive an LLD problem and independently:

1. Clarify requirements
2. Identify important entities
3. Identify responsibilities
4. Define relationships
5. Design interfaces/classes
6. Write an initial solution
7. Handle changing requirements
8. Identify design problems
9. Apply appropriate principles/patterns
10. Implement the final design in TypeScript
11. Explain my design decisions clearly

---

# 📈 Progress

### Current Stage

**Starting LLD from scratch**

### Problems Solved

**0**

### Design Patterns Learned

**0**

### SOLID Principles Learned

**0 / 5**

### Current Focus

**Building design thinking**

---

## ⭐ Core Principle

> **Don't memorize the solution. Understand why the design evolved into the solution.**
