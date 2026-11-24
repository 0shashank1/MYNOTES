

# C++ Competitive Programming: `multiset<T>`

## 1. Short notes (quick revision)

### Definition

A `multiset<T>` is an STL associative container that stores elements in sorted order and allows duplicates. By default, elements are sorted in ascending order.

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    multiset<int> ms;

    ms.insert(10);
    ms.insert(20);
    ms.insert(10);
    ms.insert(30);

    for (int x : ms)
        cout << x << " ";
}
```

Output:

```
10 10 20 30
```

### Important operations

|Operation|Purpose|Complexity|
|---|---|---|
|`ms.insert(x)`|Insert an element|\(O(\log n)\)|
|`ms.emplace(x)`|Construct and insert|\(O(\log n)\)|
|`ms.find(x)`|Find one occurrence|\(O(\log n)\)|
|`ms.count(x)`|Count occurrences|\(O(\log n+c)\)|
|`ms.contains(x)`|Check existence (C++20)|\(O(\log n)\)|
|`ms.erase(x)`|Remove all occurrences of `x`|\(O(\log n+c)\)|
|`ms.erase(it)`|Remove one occurrence|Amortized \(O(1)\)|
|`ms.lower_bound(x)`|First element \(\ge x\)|\(O(\log n)\)|
|`ms.upper_bound(x)`|First element \(> x\)|\(O(\log n)\)|
|`ms.size()`|Total number of elements|\(O(1)\)|
|`ms.empty()`|Check whether empty|\(O(1)\)|
|`ms.clear()`|Remove all elements|\(O(n)\)|

Here, \(n\) is the number of elements and \(c\) is the number of occurrences of `x`.

Remember: `multiset` allows duplicates, maintains sorted order, and does not support indexing.

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
multiset<int> ms;
multiset<string> names;

multiset<int> a = {5, 2, 5, 1, 2};

// Contains: {1, 2, 2, 5, 5}
```

For descending order:

```
multiset<int, greater<int>> ms = {1, 5, 3, 5, 2};

// Order: 5, 5, 3, 2, 1
```

A `multiset` is typically implemented as a balanced binary search tree, commonly a red-black tree.

### B. Inserting elements

```
multiset<int> ms;

ms.insert(10);
ms.insert(10);
ms.insert(20);
ms.emplace(10);

// Contains three 10s and one 20
```

Unlike `set`, repeated insertions are not ignored. Every insertion adds another occurrence.

### C. Accessing and iterating

```
multiset<int> ms = {30, 10, 20, 10};

for (int x : ms)
    cout << x << " ";
```

Output:

```
10 10 20 30
```

Iterator-based traversal:

```
for (auto it = ms.begin(); it != ms.end(); ++it)
    cout << *it << " ";
```

A multiset has no random indexing, so `ms[0]` is invalid. Its elements cannot be modified through iterators; erase an element and insert its replacement if needed.

### D. Counting duplicate elements

```
multiset<int> ms = {1, 2, 2, 2, 3, 3};

cout << ms.count(2);  // 3
cout << ms.count(3);  // 2
cout << ms.count(9);  // 0
```

`count(x)` returns the number of occurrences of `x`.

Its complexity is \(O(\log n+c)\), where \(c\) is the number of matching elements.

### E. Erasing elements — an important interview concept

Remove all occurrences of a value:

```
multiset<int> ms = {10, 10, 10, 20};

ms.erase(10);

// Result: {20}
```

Remove only one occurrence:

```
multiset<int> ms = {10, 10, 10, 20};

auto it = ms.find(10);

if (it != ms.end())
    ms.erase(it);

// Result: {10, 10, 20}
```

This is one of the most common sources of bugs in competitive programming.

- `ms.erase(x)` removes every occurrence of `x`.
    
- `ms.erase(ms.find(x))` removes just one occurrence, provided `x` exists.
    

### F. `lower_bound()` and `upper_bound()`

```
multiset<int> ms = {10, 20, 20, 20, 30, 40};

auto it1 = ms.lower_bound(20);
auto it2 = ms.upper_bound(20);
```

- `lower_bound(20)` points to the first `20`.
    
- `upper_bound(20)` points to `30`.
    

To count the occurrences of a value using iterators:

```
int cnt = distance(
    ms.lower_bound(20),
    ms.upper_bound(20)
);
```

This returns `3`, but `distance()` can take linear time because multiset iterators are bidirectional. Use `ms.count(20)` when you only need the frequency.

### G. Finding the smallest element greater than or equal to `x`

```
multiset<int> ms = {10, 20, 20, 30, 40};

int x = 25;
auto it = ms.lower_bound(x);

if (it != ms.end())
    cout << *it;  // 30
else
    cout << "No suitable element";
```

This is useful for greedy matching, scheduling, and resource allocation.

### H. Finding and removing the smallest element greater than or equal to `x`

```
multiset<int> ms = {10, 20, 30, 40};

int x = 25;
auto it = ms.lower_bound(x);

if (it != ms.end()) {
    cout << *it;  // 30
    ms.erase(it);
}
```

This pattern is useful when each available value can be used only once, such as assigning the smallest available resource that satisfies a requirement.

### I. Finding the largest element less than or equal to `x`

```
multiset<int> ms = {10, 20, 30, 40};

int x = 25;
auto it = ms.upper_bound(x);

if (it != ms.begin()) {
    --it;
    cout << *it;  // 20
}
```

Using `upper_bound(x)` and stepping backward finds the largest element less than or equal to `x`, if one exists.

### J. `multiset` vs. `set` vs. `unordered_multiset`

|Feature|`set`|`multiset`|`unordered_multiset`|
|---|---|---|---|
|Duplicates|No|Yes|Yes|
|Sorted order|Yes|Yes|No|
|Average lookup|\(O(\log n)\)|\(O(\log n)\)|\(O(1)\)|
|Worst-case lookup|\(O(\log n)\)|\(O(\log n)\)|\(O(n)\)|
|`lower_bound()`|Yes|Yes|No|
|Indexing|No|No|No|

Use `multiset` when you need duplicates and ordered operations. Use `unordered_multiset` when duplicates are needed but sorted order is unnecessary.

## 3. Complexity reference

|Operation|Complexity|
|---|---|
|Insert one element|\(O(\log n)\)|
|Find one occurrence|\(O(\log n)\)|
|Erase by iterator|Amortized \(O(1)\)|
|Erase all occurrences by key|\(O(\log n+c)\)|
|`lower_bound()` / `upper_bound()`|\(O(\log n)\)|
|`size()` / `empty()`|\(O(1)\)|
|Traverse all elements|\(O(n)\)|

## 4. Competitive programming patterns

Pattern 1 — Keep values sorted with duplicates

```
multiset<int> ms;

for (int x : a)
    ms.insert(x);
```

Useful for dynamically maintaining sorted values.

Pattern 2 — Remove exactly one occurrence

```
auto it = ms.find(x);

if (it != ms.end())
    ms.erase(it);
```

Essential when duplicate values must be preserved.

Pattern 3 — Greedy resource allocation

```
auto it = ms.lower_bound(required);

if (it != ms.end()) {
    int chosen = *it;
    ms.erase(it);
}
```

Selects and consumes the smallest available value that meets a minimum requirement.

Pattern 4 — Get minimum and maximum

```
if (!ms.empty()) {
    int mn = *ms.begin();
    int mx = *ms.rbegin();
}
```

Both minimum and maximum access take \(O(1)\) time.

## 5. Final revision checklist

- `multiset` stores sorted elements and permits duplicates.
    
- `insert(x)` adds another occurrence of `x`.
    
- `count(x)` returns the frequency.
    
- `erase(x)` removes all occurrences.
    
- `erase(iterator)` removes only one occurrence.
    
- `lower_bound()` and `upper_bound()` support ordered queries.
    
- Insertion, search, and bound operations take \(O(\log n)\).
    
- No indexing is supported.
    

One-line revision: `multiset<T>` is the right STL container when you need to maintain a sorted collection of duplicate values while efficiently inserting, searching, and deleting elements.