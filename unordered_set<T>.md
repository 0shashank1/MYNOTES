
# C++ Competitive Programming: `unordered_set<T>`

Here are short revision notes and detailed notes on `unordered_set` in C++, covering hashing, unique elements, STL operations, complexity, and common competitive programming patterns.

## 1. Short notes (quick revision)

### Definition

An `unordered_set<T>` is an STL container that stores unique elements using a hash table. Unlike `set`, it does not maintain elements in sorted order. Insertion, lookup, and deletion are typically \(O(1)\) on average.

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    unordered_set<int> s;

    s.insert(30);
    s.insert(10);
    s.insert(20);
    s.insert(10);  // Duplicate ignored

    cout << s.count(20);  // 1
    cout << s.count(50);  // 0
}
```

### Important operations

|Operation|Purpose|Average complexity|
|---|---|---|
|`s.insert(x)`|Insert an element|\(O(1)\)|
|`s.emplace(x)`|Construct and insert|\(O(1)\)|
|`s.find(x)`|Search for an element|\(O(1)\)|
|`s.count(x)`|Check existence (0 or 1)|\(O(1)\)|
|`s.contains(x)`|Check existence (C++20)|\(O(1)\)|
|`s.erase(x)`|Delete an element|\(O(1)\)|
|`s.size()`|Number of unique elements|\(O(1)\)|
|`s.empty()`|Check whether empty|\(O(1)\)|
|`s.clear()`|Remove all elements|\(O(n)\)|
|`s.reserve(n)`|Reserve capacity for elements|Average \(O(n)\) when rehashing is needed|

Worst-case lookup, insertion, and deletion can be \(O(n)\) because of hash collisions.

### Common competitive programming uses

- Remove duplicates
    
- Check whether an element exists
    
- Find duplicate elements in an array
    
- Track visited states
    
- Solve membership queries
    
- Optimize average-case lookup compared with `set`
    

Remember: `unordered_set` stores unique values, has no sorted order, and does not support indexing such as `s[0]`.

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
unordered_set<int> s;
unordered_set<string> names;

unordered_set<int> a = {10, 20, 30, 20};

// Stores unique values: 10, 20, 30
```

Initialize from a vector:

```
vector<int> v = {1, 2, 2, 3, 4, 4};

unordered_set<int> s(v.begin(), v.end());
```

The duplicate values are automatically discarded.

### B. Insertion

```
unordered_set<int> s;

s.insert(10);
s.insert(20);
s.insert(10);  // Ignored
s.emplace(30);
```

To check whether insertion succeeded:

```
auto [it, inserted] = s.insert(20);

if (inserted)
    cout << "Inserted";
else
    cout << "Already exists";
```

Structured bindings require C++17.

### C. Searching for an element

```
unordered_set<int> s = {10, 20, 30};

auto it = s.find(20);

if (it != s.end())
    cout << "Found";
else
    cout << "Not found";
```

In C++20 and later, you can use:

```
if (s.contains(20))
    cout << "Found";
```

`find()` returns an iterator to the element or `s.end()` if it is absent.

### D. Deleting elements

Erase by value:

```
s.erase(20);
```

Erase using an iterator:

```
auto it = s.find(30);

if (it != s.end())
    s.erase(it);
```

Remove all elements:

```
s.clear();
```

Erasing by value has average \(O(1)\) complexity and worst-case \(O(n)\).

### E. Iterating through an unordered set

```
unordered_set<int> s = {10, 20, 30};

for (int x : s) {
    cout << x << " ";
}
```

The output order is unspecified. Do not assume ascending order or insertion order.

If sorted output is required:

```
vector<int> v(s.begin(), s.end());
sort(v.begin(), v.end());
```

This takes \(O(n\log n)\) time for sorting.

### F. Detecting duplicates in an array

```
bool containsDuplicate(vector<int>& a) {
    unordered_set<int> seen;

    for (int x : a) {
        if (seen.count(x))
            return true;

        seen.insert(x);
    }

    return false;
}
```

Example:

```
Array: [1, 2, 3, 2]
Output: true
```

The algorithm stops when it finds a repeated element.

- Average time: \(O(n)\)
    
- Auxiliary space: \(O(n)\)
    

### G. Finding distinct elements

```
vector<int> a = {1, 2, 2, 3, 3, 4};

unordered_set<int> s(a.begin(), a.end());

cout << s.size();  // 4
```

This is useful when you only need the number of distinct values.

If you also need sorted unique values, use `set<int>` or sort and deduplicate a vector.

### H. `reserve()` and `load_factor()`

For large inputs, reserving capacity may reduce rehashing.

```
unordered_set<int> s;

s.reserve(100000);

for (int i = 0; i < 100000; i++)
    s.insert(i);
```

Other useful functions:

```
s.bucket_count();
s.load_factor();
s.max_load_factor();
```

You can set the load-factor threshold before reserving:

```
s.max_load_factor(0.7);
s.reserve(100000);
```

This may trade additional memory for fewer collisions. `reserve()` does not insert elements; it requests enough buckets for the specified number of elements under the current maximum load factor.

### I. Custom types

For standard types such as `int` and `string`, built-in hash support is available.

For custom types, you may need to provide a hash function and equality comparison.

Example using `pair<int,int>`:

```
struct PairHash {
    size_t operator()(const pair<int, int>& p) const {
        size_t h1 = hash<int>{}(p.first);
        size_t h2 = hash<int>{}(p.second);

        return h1 ^ (h2 + 0x9e3779b9
                     + (h1 << 6) + (h1 >> 2));
    }
};

unordered_set<pair<int, int>, PairHash> s;

s.insert({1, 2});
s.insert({3, 4});

cout << s.count({1, 2});  // 1
```

This pattern is useful for grid coordinates, visited states, and coordinate-based problems.

### J. `set` vs. `unordered_set` vs. `multiset`

|Feature|`set`|`unordered_set`|`multiset`|
|---|---|---|---|
|Unique elements|Yes|Yes|No|
|Sorted order|Yes|No|Yes|
|Average lookup|\(O(\log n)\)|\(O(1)\)|\(O(\log n)\)|
|Worst-case lookup|\(O(\log n)\)|\(O(n)\)|\(O(\log n)\)|
|`lower_bound()`|Yes|No|Yes|
|Indexing|No|No|No|

Choose:

- `set` for sorted unique values and ordered queries.
    
- `unordered_set` for fast average-case membership checks.
    
- `multiset` when duplicates must be retained in sorted order.
    

## 3. Complexity reference

|Operation|Average|Worst case|
|---|---|---|
|Insert|\(O(1)\)|\(O(n)\)|
|Find|\(O(1)\)|\(O(n)\)|
|Erase by value|\(O(1)\)|\(O(n)\)|
|Check existence|\(O(1)\)|\(O(n)\)|
|Traverse all elements|\(O(n)\)|\(O(n)\)|
|Build from \(n\) elements|\(O(n)\) average|\(O(n^2)\) worst case|

## 4. Competitive programming patterns

Pattern 1 — Fast membership check

```
unordered_set<int> s(a.begin(), a.end());

if (s.count(x))
    cout << "Present";
```

Useful for repeated existence queries.

Pattern 2 — Remove duplicates

```
unordered_set<int> unique(a.begin(), a.end());
```

Creates a collection of distinct values, with unspecified iteration order.

Pattern 3 — Track visited states

```
unordered_set<int> visited;

if (visited.insert(x).second) {
    // x was not previously present
}
```

Combines insertion and membership checking in one operation.

Pattern 4 — Find common elements

```
unordered_set<int> s(a.begin(), a.end());

for (int x : b) {
    if (s.count(x))
        cout << x << " ";
}
```

Prints matching elements from `b`; repeated elements in `b` may be printed repeatedly.

## 5. Final revision checklist

- `unordered_set` stores unique elements using hashing.
    
- Average insertion, lookup, and deletion are \(O(1)\).
    
- Worst-case operations can be \(O(n)\).
    
- Iteration order is unspecified.
    
- Use `find()`, `count()`, or `contains()` for membership checks.
    
- Use `reserve()` to reduce rehashing for large known input sizes.
    
- Use a custom hash for unsupported key types such as `pair<int,int>`.
    
- Prefer `set` if you need sorted elements or `lower_bound()`.
    

One-line revision: `unordered_set<T>` is the go-to STL container for fast average-case membership testing, duplicate detection, and storing unique values without requiring sorted order.