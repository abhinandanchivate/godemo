

---

# MinHeap Usage in Parking Lot System

---

##  Opening Context

“Before we talk about parking cars, we must answer a fundamental question:

👉 When a driver arrives, **which parking slot should we give?**

Most parking systems follow a simple rule:

👉 Always assign the **lowest numbered free slot**.

Slot 1 before Slot 2, Slot 2 before Slot 3, and so on.

So our real problem becomes:

👉 How do we always know the **smallest available slot quickly**?”

---

## 🚨 Naive Approach (What We Avoid)

“If we store slots in an array:

```
[occupied, occupied, free, occupied, free]
```

To find a free slot, we must loop:

```
for i := 0 to N
```

That is:

👉 O(N) time for every parking.

If N = 100,000 slots, this is slow.”

---

## ✅ Our Chosen Solution

“We use a **MinHeap** that stores only **free slot numbers**.

A MinHeap always gives us:

👉 The smallest element at the top.

So:

```
freeSlots heap = [1,2,3,4,5]
Top = 1
```

Slot 1 is instantly available.”

---

---

# 📄 Introducing minheap.go

“We now implement our own MinHeap so Go’s heap library can work with it.”

---

```go
type MinHeap []int
```

“This line means:

MinHeap is simply a slice of integers.

Each integer represents:

👉 A free parking slot number.”

Example:

```
MinHeap = [1,4,7]
```

Slots 1,4,7 are free.

---

---

## 📌 Why Do We Write Methods?

“Go’s `container/heap` package does not know how our data should behave.

So we must teach it using 5 methods.”

---

---

### 1️⃣ Len()

```go
func (h MinHeap) Len() int {
    return len(h)
}
```

“This tells heap:

👉 How many items exist.

If heap contains:

```
[1,2,3]
```

Len() returns 3.”

---

---

### 2️⃣ Less(i, j)

```go
func (h MinHeap) Less(i, j int) bool {
    return h[i] < h[j]
}
```

“This defines ordering.

If element at i is smaller than element at j → higher priority.

Because we used `<`, smallest number wins.

Therefore:

👉 This becomes a **MinHeap**.

If we wrote:

```
h[i] > h[j]
```

It would become MaxHeap.”

---

---

### 3️⃣ Swap(i, j)

```go
func (h MinHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}
```

“Heap algorithm rearranges elements frequently.

So we provide swap ability.”

---

---

### 4️⃣ Push(x)

```go
func (h *MinHeap) Push(x interface{}) {
    *h = append(*h, x.(int))
}
```

“Push means: add new free slot.

Important details:

* Pointer receiver → because we modify slice
* x is interface{} → we typecast to int
* Append into slice

Example:

Before:

```
[2,3,4]
```

Push(1)

After append:

```
[2,3,4,1]
```

Heap library will reorder internally.”

---

---

### 5️⃣ Pop()

```go
func (h *MinHeap) Pop() interface{} {
    old := *h
    n := len(old)
    val := old[n-1]
    *h = old[:n-1]
    return val
}
```

“Pop removes and returns last element.

Important concept:

👉 Heap library moves smallest element to the end **before** calling this Pop.

So Pop actually returns smallest slot.”

Example:

Heap arranges:

```
[4,2,3,1]
```

Pop returns:

```
1
```

---

---

# ✅ What We Have Achieved

We have now created a structure that:

✔ Stores integers
✔ Always knows smallest value
✔ Supports fast insert & remove

---

---

# 🔁 How ParkingLot Uses MinHeap

---

## During Parking Lot Creation

```go
for i := 1; i <= size; i++ {
    heap.Push(h, i)
}
```

Spoken:

“We push all slot numbers into heap.

If size = 5:

Heap becomes:

````
[1,2,3,4,5]
```”

---

---

## When Car Parks

```go
slot := heap.Pop(p.freeSlots)
````

Spoken:

“Give me smallest free slot.

Heap returns:

```
1
```

Now heap becomes:

````
[2,3,4,5]
```”

---

---

## When Car Unparks

```go
heap.Push(p.freeSlots, slot)
````

Spoken:

“If car leaves slot 1, push 1 back.

Heap becomes:

````
[1,2,3,4,5]
```”

---

---

# 🧠 Mental Model for Students

“Think of a **bag of numbered tokens**.

- Each token = free slot
- You always pick smallest token
- When someone leaves, you drop token back

That bag is MinHeap.”

---

---

# ⚡ Performance Benefit

| Operation | Time |
|---------|------|
| Push | O(log N) |
| Pop | O(log N) |
| Get smallest | O(1) |

No scanning.

---

---

# 🎯 Why MinHeap Is Perfect Here

1) Deterministic (always lowest slot)  
2) Fast  
3) Simple  
4) Scales well  

---

---

# 🏁 Closing Statement

“In our parking lot system, MinHeap is the **brain of slot allocation**.

ParkingLot decides rules.  
MinHeap decides **which slot number**.

They work together.”

---


