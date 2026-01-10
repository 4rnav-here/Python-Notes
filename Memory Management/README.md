## 📌 What Is Memory Management?

**Memory management** is the process of:

* Allocating memory to objects when they are created
* Deallocating (freeing) memory when objects are no longer needed

In Python, **memory management is automatic**.
You do **not** manually allocate or free memory like in C/C++.

Python handles memory using:

1. **Reference Counting**
2. **Garbage Collection**
3. **Private Heap & Allocators**
4. **Stack and Heap separation**

---

## 🧩 Python’s Memory Architecture (Big Picture)

```
+-------------------+
|   Stack Memory    |  ← variable names, function calls, references
+-------------------+
|   Heap Memory     |  ← actual objects (lists, dicts, strings, etc.)
+-------------------+
| Python Private Heap (Managed by Python)
+-------------------+
| OS Memory
+-------------------+
```

👉 **Important:**
Variables do **not** store values.
They store **references (addresses)** to objects in heap memory.

---

## 🏗️ Python Memory Management Components

Python internally uses **three major components**:

### 1️⃣ Private Heap

* Stores **all Python objects**
* Not accessible directly by the programmer
* Fully managed by Python’s memory manager

---

### 2️⃣ Raw Memory Allocator

* Communicates with the **Operating System**
* Reserves large chunks of memory
* Used for **large objects** and arenas

---

### 3️⃣ Object-Specific Allocators

* Optimize memory for specific object types
* Examples:

  * Integers
  * Strings
  * Tuples
  * Lists
  * Dictionaries

---

## 🧠 Stack vs Heap Memory (Very Important)

### 🔹 Stack Memory (Fast, Temporary)

Used for:

* Function calls
* Local variables
* References (not actual objects)

Characteristics:

* LIFO (Last In, First Out)
* Automatically freed
* Very fast
* Limited in size

Example:

```python
def my_function():
    x = 5
    y = True
    z = "Hello"
    return x, y, z
```

* `x`, `y`, `z` → stored on **stack**
* Actual objects (`5`, `True`, `"Hello"`) → stored in **heap**

Once function ends → stack memory is cleared.

---

### 🔹 Heap Memory (Flexible, Persistent)

Used for:

* Objects (lists, dicts, strings, classes)
* Shared across functions
* Managed by Garbage Collector

Example:

```python
a = [0] * 10
```

* List object → heap
* Variable `a` → stack (stores reference)

---

## 🔗 Reference Counting (Core of Python Memory Management)

### What Is Reference Counting?

Each Python object keeps track of:

> **How many references point to it**

This count is called the **reference count**.

---

### How It Works

```python
a = [1, 2, 3]   # ref count = 1
b = a           # ref count = 2
c = a           # ref count = 3

del b           # ref count = 2
del c           # ref count = 1
del a           # ref count = 0 → object destroyed
```

Once reference count becomes **zero**, Python:

* Immediately frees the object
* Reclaims memory

---

### Checking Reference Count

```python
import sys

a = [1, 2, 3]
print(sys.getrefcount(a))  # includes temporary reference
```

⚠️ `getrefcount()` always shows **+1 extra reference** due to function call.

---

## ✅ Advantages of Reference Counting

* Immediate memory cleanup
* Deterministic object destruction
* Simple and predictable
* Excellent for resource management (files, sockets)

---

## ❌ Limitations of Reference Counting

### ❗ Circular References Problem

```python
class Node:
    def __init__(self, value):
        self.next = None

a = Node(1)
b = Node(2)

a.next = b
b.next = a
```

Even after:

```python
del a
del b
```

Objects are **not freed**, because:

* `a` references `b`
* `b` references `a`

Reference count never reaches zero.

➡️ This is where **Garbage Collection** comes in.

---

## 🗑️ Garbage Collection (GC)

### What Is Garbage Collection?

Garbage Collection is a **backup system** that:

* Detects unreachable objects
* Breaks reference cycles
* Frees memory

Python uses:

* **Reference Counting** (primary)
* **Cyclic Garbage Collector** (secondary)

---

## 🔁 How Python GC Works

### 1️⃣ Generation-Based Collection

Objects are grouped into **3 generations**:

| Generation | Meaning            |
| ---------- | ------------------ |
| Gen 0      | New objects        |
| Gen 1      | Survived once      |
| Gen 2      | Long-lived objects |

> Most objects die young → optimized cleanup

---

### 2️⃣ Mark-and-Sweep Algorithm

Steps:

1. GC starts from **root objects**
2. Marks all reachable objects
3. Sweeps (deletes) unmarked ones

---

### 3️⃣ GC Thresholds

GC runs when:

```python
allocations - deallocations > threshold
```

View thresholds:

```python
import gc
print(gc.get_threshold())
```

---

## 🛠️ Controlling Garbage Collection

### Enable / Disable GC

```python
import gc

gc.disable()
# critical code
gc.enable()
```

---

### Force GC Manually

```python
gc.collect()
```

---

### Adjust GC Thresholds

```python
gc.set_threshold(700, 10, 10)
```

---

### Inspect GC State

```python
gc.get_count()
gc.get_stats()
```

---

## 🧬 Weak References (Advanced but Important)

Weak references:

* Do **not increase reference count**
* Prevent memory leaks
* Common in caches

```python
import weakref

class MyClass:
    pass

obj = MyClass()
weak = weakref.ref(obj)

del obj
print(weak())  # None
```

---

## ⚙️ Python Memory Allocators (Under the Hood)

### Pymalloc (Small Object Optimizer)

* Used for objects **< 512 bytes**
* Faster than OS allocation
* Reduces fragmentation

---

### Arenas, Pools, and Blocks

| Level | Size     | Purpose            |
| ----- | -------- | ------------------ |
| Arena | 256 KB   | Large memory chunk |
| Pool  | 4 KB     | Same-sized objects |
| Block | Variable | Actual object      |

This hierarchy:

* Improves speed
* Reduces fragmentation
* Reuses memory efficiently

---

## 🔁 Object Interning (Memory Optimization)

Python reuses certain immutable objects.

### Small Integers (-5 to 256)

```python
x = 10
y = 10
print(id(x) == id(y))  # True
```

Changing value creates new object:

```python
x += 1
print(id(x) == id(y))  # False
```

---

## 🧠 Memory Management in Practice

### ❌ Inefficient

```python
for i in range(1000):
    temp = [i]
```

### ✅ Efficient

```python
temp = []
for i in range(1000):
    temp.append(i)
    temp.clear()
```

---

### Use Generators

```python
def gen():
    for i in range(10000):
        yield i
```

---

### Use Context Managers

```python
with open("file.txt") as f:
    data = f.read()
```

---

## 🎯 Common Interview Questions & Answers

### Q1. Where are Python objects stored?

👉 Heap memory

---

### Q2. Where are variables stored?

👉 Stack (as references)

---

### Q3. Does Python have manual memory management?

👉 No

---

### Q4. What causes memory leaks in Python?

👉 Circular references + external resources

---

### Q5. Difference between GC and reference counting?

👉 Reference counting is immediate, GC handles cycles

---

### Q6. Why is Python slower than C?

👉 Dynamic typing, object overhead, memory indirection

---

### Q7. Why do lists share changes when assigned?

👉 Because assignment copies references, not objects

---

## 🧠 Final Mental Model (Very Important)

> **Variables → References → Objects → Heap → Managed by Python**

Once this clicks:

* Lists
* Mutability
* Performance
* Bugs
* Memory leaks
  all start making sense.

---
