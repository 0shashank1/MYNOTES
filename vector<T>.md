
# C++ Competitive Programming: `vector<T>`

## 1. Short notes (quick revision)

### Definition

A `vector<T>` is a dynamic array from the C++ STL that can grow or shrink at runtime. It stores elements contiguously in memory and supports fast random access.

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30};

    v.push_back(40);
    cout << v[0];  // 10
    cout << v.back(); // 40
}
```

### Important operations

|Operation|Purpose|Complexity|
|---|---|---|
|`v.push_back(x)`|Add element at end|Amortized \(O(1)\)|
|`v.emplace_back(x)`|Construct element at end|Amortized \(O(1)\)|
|`v.pop_back()`|Remove last element|\(O(1)\)|
|`v[i]`|Access by index|\(O(1)\)|
|`v.at(i)`|Bounds-checked access|\(O(1)\)|
|`v.front()`|First element|\(O(1)\)|
|`v.back()`|Last element|\(O(1)\)|
|`v.size()`|Number of elements|\(O(1)\)|
|`v.empty()`|Check whether empty|\(O(1)\)|
|`v.clear()`|Remove all elements|\(O(n)\)|
|`v.resize(n)`|Change the size|\(O(n)\) in general|
|`v.reserve(n)`|Request capacity of at least `n`|May be \(O(n)\)|
|`v.insert()`|Insert at a position|\(O(n)\) in general|
|`v.erase()`|Erase at a position|\(O(n)\) in general|

### Sorting and searching

```
sort(v.begin(), v.end());                 // Ascending
sort(v.begin(), v.end(), greater<int>()); // Descending

reverse(v.begin(), v.end());

auto it = find(v.begin(), v.end(), 20);

bool exists = binary_search(v.begin(), v.end(), 20);
```

Important: `binary_search()` requires the vector to be sorted according to the search ordering.

### Common uses

- Arrays with dynamic size
    
- Sorting and searching
    
- Prefix sums and sliding windows
    
- Graph adjacency lists
    
- Dynamic programming
    
- Storing pairs and custom structures
    
- Two-dimensional matrices
    

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
vector<int> a;                  // Empty vector
vector<int> b(5);               // Five zeros
vector<int> c(5, 10);           // Five elements, all 10
vector<int> d = {1, 2, 3, 4};   // Initializer list
vector<int> e(d);               // Copy of d
```

Other useful declarations:

```
vector<long long> a;
vector<char> chars;
vector<string> words;
vector<pair<int, int>> points;
vector<bool> visited(10, false);
```

For competitive programming, `vector<long long>` is useful when values or prefix sums may exceed the range of a 32-bit integer.

### B. Adding and removing elements

```
vector<int> v;

v.push_back(10);
v.push_back(20);
v.emplace_back(30);

// v = {10, 20, 30}

v.pop_back();

// v = {10, 20}
```

`push_back()` appends an element, while `pop_back()` removes the last element.

Calling `pop_back()` on an empty vector is invalid.

### C. Accessing elements

```
vector<int> v = {10, 20, 30, 40};

cout << v[0];      // 10
cout << v.at(1);   // 20
cout << v.front(); // 10
cout << v.back();  // 40
```

- `v[i]` provides fast access but does not check bounds.
    
- `v.at(i)` checks bounds and throws `std::out_of_range` if the index is invalid.
    
- Valid indices range from `0` to `v.size() - 1`.
    

### D. Iterating through a vector

Using an index:

```
for (int i = 0; i < (int)v.size(); i++) {
    cout << v[i] << " ";
}
```

Using a range-based loop:

```
for (int x : v) {
    cout << x << " ";
}
```

Using a reference to modify elements:

```
for (int& x : v) {
    x *= 2;
}
```

Using iterators:

```
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << " ";
}
```

Use `const auto&` when you want to read elements without copying them, especially for strings or large objects.

### E. Sorting a vector

```
vector<int> v = {5, 2, 8, 1, 3};

sort(v.begin(), v.end());

// {1, 2, 3, 5, 8}
```

Descending order:

```
sort(v.begin(), v.end(), greater<int>());
```

Custom comparator:

```
vector<pair<int, int>> v = {
    {2, 5}, {1, 8}, {2, 3}
};

sort(v.begin(), v.end(), [](auto a, auto b) {
    if (a.first != b.first)
        return a.first < b.first;

    return a.second > b.second;
});
```

This sorts by the first element ascending and, for ties, the second element descending.

Complexity: \(O(n\log n)\).

### F. Searching in a vector

Linear search:

```
auto it = find(v.begin(), v.end(), 5);

if (it != v.end()) {
    cout << "Found at index "
         << it - v.begin();
}
```

Time complexity: \(O(n)\).

Binary search:

```
sort(v.begin(), v.end());

bool found = binary_search(
    v.begin(), v.end(), 5
);
```

Time complexity: \(O(\log n)\) comparisons for a vector.

Find the first element greater than or equal to a value:

```
auto it = lower_bound(v.begin(), v.end(), 5);
```

Find the first element strictly greater than a value:

```
auto it = upper_bound(v.begin(), v.end(), 5);
```

Both require a range sorted according to the relevant ordering.

### G. Insertion and deletion

Insert at an index:

```
vector<int> v = {10, 20, 30};

v.insert(v.begin() + 1, 15);

// {10, 15, 20, 30}
```

Erase one element:

```
v.erase(v.begin() + 1);
```

Erase a range:

```
v.erase(v.begin() + 1, v.begin() + 3);
```

Remove all occurrences of a value using the erase-remove idiom:

```
v.erase(
    remove(v.begin(), v.end(), 20),
    v.end()
);
```

In C++20, you can use:

```
erase(v, 20);
```

Insertion and deletion in the middle typically take \(O(n)\), because subsequent elements must be shifted.

### H. `size()`, `capacity()`, `resize()`, and `reserve()`

These four concepts are important for technical interviews and contests.

```
vector<int> v;

v.reserve(100);  // Capacity at least 100
v.push_back(10); // Size is 1
v.resize(5);     // Size is 5
```

|Function|Meaning|
|---|---|
|`size()`|Number of existing elements|
|`capacity()`|Number of elements that fit without reallocation|
|`resize(n)`|Changes the number of elements|
|`reserve(n)`|Requests storage capacity without changing size|

`reserve()` is useful when you know the approximate number of elements in advance, because it can reduce reallocations.

### I. Two-dimensional vectors

A 2D vector can represent a matrix or grid.

```
int n = 3, m = 4;

vector<vector<int>> grid(
    n, vector<int>(m, 0)
);

grid[1][2] = 7;
cout << grid[1][2];  // 7
```

Iterate through it:

```
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        cout << grid[i][j] << " ";
    }
    cout << '\n';
}
```

For a grid with \(n\) rows and \(m\) columns, initializing and traversing all cells takes \(O(nm)\) time.

### J. Prefix sums using vectors

Prefix sums let you answer range-sum queries efficiently.

```
vector<int> a = {2, 4, 1, 7};
int n = a.size();

vector<long long> pref(n + 1, 0);

for (int i = 0; i < n; i++) {
    pref[i + 1] = pref[i] + a[i];
}

// Sum of a[l ... r], inclusive:
int l = 1, r = 3;

long long sum = pref[r + 1] - pref[l];
cout << sum;  // 12
```

- Preprocessing: \(O(n)\)
    
- Each range-sum query: \(O(1)\)
    
- Extra space: \(O(n)\)
    

Use `long long` for prefix sums when the total may overflow `int`.

### K. Important pitfalls

1. Out-of-bounds access: `v[v.size()]` is invalid.
    
2. Empty vector: Check `!v.empty()` before using `front()` or `back()`.
    
3. Iterator invalidation: Insertion or appending can invalidate iterators and references if reallocation occurs. Erasing invalidates iterators and references at or after the erased position.
    
4. Size versus capacity: `reserve(100)` does not create 100 elements.
    
5. Unsigned size: `v.size()` returns an unsigned type. Mixing it carelessly with negative integers can cause bugs.
    
6. Repeated front insertion: `v.insert(v.begin(), x)` takes \(O(n)\); use `deque` if frequent front insertions are required.
    

## 3. Complexity reference

|Operation|Time complexity|
|---|---|
|Random access|\(O(1)\)|
|`push_back()`|Amortized \(O(1)\)|
|`pop_back()`|\(O(1)\)|
|`find()`|\(O(n)\)|
|`sort()`|\(O(n\log n)\)|
|`binary_search()`|\(O(\log n)\) comparisons|
|`lower_bound()`|\(O(\log n)\) comparisons|
|Insert or erase in middle|\(O(n)\)|
|Traverse all elements|\(O(n)\)|

## 4. Competitive programming patterns

Pattern 1 — Dynamic array

```
vector<int> a(n);
for (int& x : a)
    cin >> x;
```

Use when the number of elements is known from input.

Pattern 2 — Remove duplicates from a sorted vector

```
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());
```

Sorts first, then removes adjacent duplicates. Total time is \(O(n\log n)\).

Pattern 3 — Graph adjacency list

```
vector<vector<int>> adj(n);

adj[u].push_back(v);
adj[v].push_back(u); // Undirected graph
```

Stores edges efficiently for BFS and DFS.

Pattern 4 — Frequency counting

```
vector<int> freq(1001, 0);

for (int x : a)
    freq[x]++;
```

Works when values are known to lie between 0 and 1000.

Final takeaway: Master vector initialization, iterators, sorting, binary search, `lower_bound()`, erase-remove, and prefix sums. These are foundational STL techniques for competitive programming.