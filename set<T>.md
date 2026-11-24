
# C++ Competitive Programming: `set<T>`

## 1. Short notes (quick revision)

### Definition

A `set<T>` is an STL associative container that stores unique elements in sorted order. Duplicate insertions are ignored. By default, elements are sorted in ascending order.

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    set<int> s;

    s.insert(30);
    s.insert(10);
    s.insert(20);
    s.insert(10);  // Duplicate ignored

    for (int x : s)
        cout << x << " ";
}
```

Output:

```
10 20 30
```

### Important operations

|Operation|Purpose|Complexity|
|---|---|---|
|`s.insert(x)`|Insert an element|\(O(\log n)\)|
|`s.emplace(x)`|Construct and insert|\(O(\log n)\)|
|`s.find(x)`|Find an element|\(O(\log n)\)|
|`s.count(x)`|Check existence (0 or 1)|\(O(\log n)\)|
|`s.contains(x)`|Check existence (C++20)|\(O(\log n)\)|
|`s.erase(x)`|Remove an element by value|\(O(\log n)\)|
|`s.erase(it)`|Remove by iterator|Amortized \(O(1)\)|
|`s.lower_bound(x)`|First element \(\ge x\)|\(O(\log n)\)|
|`s.upper_bound(x)`|First element \(> x\)|\(O(\log n)\)|
|`s.size()`|Number of elements|\(O(1)\)|
|`s.empty()`|Check if empty|\(O(1)\)|
|`s.clear()`|Remove all elements|\(O(n)\)|

Here, \(n\) is the number of elements.

### Common uses in competitive programming

- Remove duplicates from data
    
- Maintain sorted unique values
    
- Find the next greater or smallest eligible element
    
- Maintain dynamic ordered collections
    
- Coordinate compression
    
- Solve successor/predecessor problems
    

Remember: A `set` is sorted and unique, but it does not support indexing like `vector`.

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
set<int> s;
set<string> names;

set<int> a = {5, 2, 8, 2, 1};

// Stores: {1, 2, 5, 8}
```

You can also declare a set in descending order:

```
set<int, greater<int>> s = {1, 5, 3, 2};

// Order: 5, 3, 2, 1
```

A `set` is typically implemented using a balanced binary search tree, commonly a red-black tree.

### B. Inserting elements

```
set<int> s;

s.insert(10);
s.insert(20);
s.insert(10);  // Ignored because 10 already exists

s.emplace(30);
```

Only one copy of each value is stored.

You can check whether insertion actually occurred:

```
auto [it, inserted] = s.insert(20);

if (inserted)
    cout << "Inserted";
else
    cout << "Already exists";
```

The structured binding syntax requires C++17.

### C. Accessing and iterating

```
set<int> s = {40, 10, 30, 20};

for (int x : s)
    cout << x << " ";
```

Output:

```
10 20 30 40
```

Iterator-based traversal:

```
for (auto it = s.begin(); it != s.end(); ++it)
    cout << *it << " ";
```

Important points:

- `*it` gives the element.
    
- Set elements cannot be modified through their iterators.
    
- `s[0]` is invalid because a set does not support indexing.
    

### D. Searching for an element

```
set<int> s = {10, 20, 30, 40};

auto it = s.find(30);

if (it != s.end())
    cout << "Found: " << *it;
else
    cout << "Not found";
```

`find()` returns an iterator to the element if found; otherwise, it returns `s.end()`.

For C++20:

```
if (s.contains(30))
    cout << "Found";
```

### E. Deleting elements

Erase by value:

```
s.erase(20);
```

Erase by iterator:

```
auto it = s.find(30);

if (it != s.end())
    s.erase(it);
```

Clear the entire set:

```
s.clear();
```

`erase(value)` returns the number of elements removed: `0` if the value was absent and `1` if it was present.

### F. `lower_bound()` and `upper_bound()`

These are among the most useful operations in competitive programming.

```
set<int> s = {10, 20, 30, 40, 50};

auto it1 = s.lower_bound(25); // 30
auto it2 = s.upper_bound(30); // 40
```

- `lower_bound(x)` returns the first element greater than or equal to `x`.
    
- `upper_bound(x)` returns the first element strictly greater than `x`.
    

Always check whether the iterator equals `s.end()` before dereferencing it.

Example: Find the smallest element greater than or equal to `x`.

```
int x = 35;

auto it = s.lower_bound(x);

if (it != s.end())
    cout << *it;  // 40
else
    cout << "No suitable element";
```

Example: Find the largest element smaller than `x`.

```
int x = 35;

auto it = s.lower_bound(x);

if (it != s.begin()) {
    --it;
    cout << *it;  // 30
}
```

This predecessor pattern is useful for ordered lookup problems.

### G. Set operations: union, intersection, and difference

You can use STL algorithms on sorted sets by writing the results to vectors or another suitable output container.

Union:

```
set<int> a = {1, 2, 3};
set<int> b = {3, 4, 5};

vector<int> result;

set_union(
    a.begin(), a.end(),
    b.begin(), b.end(),
    back_inserter(result)
);
```

Result: `{1, 2, 3, 4, 5}`.

Intersection:

```
vector<int> result;

set_intersection(
    a.begin(), a.end(),
    b.begin(), b.end(),
    back_inserter(result)
);
```

Result: `{3}`.

Difference (`a - b`):

```
vector<int> result;

set_difference(
    a.begin(), a.end(),
    b.begin(), b.end(),
    back_inserter(result)
);
```

Result: `{1, 2}`.

These algorithms run in linear time in the total input sizes. Because the inputs are sets, each input value is unique.

### H. `set` vs. `unordered_set` vs. `multiset`

|Feature|`set`|`unordered_set`|`multiset`|
|---|---|---|---|
|Unique elements|Yes|Yes|No|
|Sorted order|Yes|No|Yes|
|Average lookup|\(O(\log n)\)|\(O(1)\)|\(O(\log n)\)|
|Worst-case lookup|\(O(\log n)\)|\(O(n)\)|\(O(\log n)\)|
|`lower_bound()`|Yes|No|Yes|
|Indexing|No|No|No|
|Best use|Sorted unique values|Fast average membership checks|Sorted values with duplicates|

Use `set` when you need sorted unique values, `unordered_set` when ordering is unnecessary, and `multiset` when duplicates must be preserved.

### I. Coordinate compression using `set`

```
vector<int> a = {100, 10, 100, 50};

set<int> s(a.begin(), a.end());

vector<int> sorted_unique(s.begin(), s.end());

// sorted_unique = {10, 50, 100}
```

This removes duplicates and sorts the values.

For assigning compressed indices, use a sorted vector and `lower_bound()`:

```
for (int x : a) {
    int idx = lower_bound(
        sorted_unique.begin(),
        sorted_unique.end(),
        x
    ) - sorted_unique.begin();

    cout << idx << " ";
}
```

The compressed indices preserve the original ordering of distinct values.

### J. Important pitfalls

1. No duplicates: Inserting an existing value does not add another copy.
    
2. No indexing: `s[i]` is invalid.
    
3. No direct modification: Erase an element and insert its new value if it needs to change.
    
4. Iterator safety: Erasing an element invalidates iterators to that element, but not iterators to other elements.
    
5. Empty set: Never dereference `begin()` or `end()` without checking the relevant conditions.
    
6. Custom comparators: The comparator must provide a strict weak ordering.
    

## 3. Complexity reference

|Operation|Complexity|
|---|---|
|Insert|\(O(\log n)\)|
|Find|\(O(\log n)\)|
|Erase by value|\(O(\log n)\)|
|`lower_bound()`|\(O(\log n)\)|
|`upper_bound()`|\(O(\log n)\)|
|`size()` / `empty()`|\(O(1)\)|
|Iterate through all elements|\(O(n)\)|
|Union/intersection of two sets|\(O(n+m)\)|

## 4. Competitive programming patterns

Pattern 1 — Remove duplicates and sort

```
set<int> s(a.begin(), a.end());
```

Creates sorted unique values from a collection.

Pattern 2 — Find the smallest eligible value

```
auto it = s.lower_bound(x);

if (it != s.end())
    cout << *it;
```

Useful for successor queries and ordered allocation problems.

Pattern 3 — Find and remove an element

```
auto it = s.find(x);

if (it != s.end())
    s.erase(it);
```

Useful when elements must be dynamically maintained.

Pattern 4 — Maintain unique active values

```
set<int> active;

active.insert(x);
active.erase(y);
```

Supports ordered insertion, deletion, and lookup in logarithmic time.

Final takeaway: Master `insert()`, `find()`, `erase()`, `lower_bound()`, and `upper_bound()`. These are the core operations for solving competitive programming problems involving sorted, unique, dynamically changing values.