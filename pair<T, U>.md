# C++ Competitive Programming: `pair<T, U>`

## 1. Short notes (quick revision)

### Definition

A `pair<T, U>` is an STL container that stores exactly two values, which can have different data types.

```
#include <bits/stdc++.h>
using namespace std;

pair<int, string> p = {1, "Alice"};

cout << p.first;   // 1
cout << p.second;  // Alice
```

### Important operations

|Syntax|Purpose|
|---|---|
|`pair<int,int> p`|Declare a pair|
|`{a, b}`|Initialize a pair|
|`make_pair(a, b)`|Create a pair|
|`p.first`|Access first element|
|`p.second`|Access second element|
|`p = {x, y}`|Update both elements|
|`swap(p1, p2)`|Swap two pairs|
|`p1 == p2`|Compare both elements|
|`tie(a, b) = p`|Unpack a pair into variables (C++11+)|
|`auto p = make_pair(a, b)`|Automatically deduce types|

### Sorting rule

Pairs are compared lexicographically:

1. Compare `.first`.
    
2. If the first elements are equal, compare `.second`.
    

```
vector<pair<int, int>> v = {
    {2, 5}, {1, 9}, {2, 3}
};

sort(v.begin(), v.end());

// Result: {1,9}, {2,3}, {2,5}
```

### Common competitive programming uses

- Store `(value, index)`.
    
- Store `(distance, node)` in Dijkstra's algorithm.
    
- Store `(start, end)` for intervals.
    
- Store `(frequency, element)` for counting problems.
    
- Store coordinates `(x, y)`.
    
- Use `vector<pair<int,int>>` for adjacency lists.
    

Remember: `first` and `second` are members, not functions. Write `p.first`, not `p.first()`.

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
pair<int, int> p1;
pair<int, string> p2;
pair<long long, int> p3;

pair<int, int> p4 = {10, 20};
pair<int, int> p5(10, 20);
pair<int, int> p6 = make_pair(10, 20);
auto p7 = make_pair(10, 20);
```

The types of the first and second elements do not need to be the same.

```
pair<int, double> p = {5, 3.14};
pair<string, int> student = {"Rahul", 95};
```

In modern C++, you can also use class template argument deduction:

```
pair p = {10, 20};  // pair<int, int>
```

### B. Accessing and modifying elements

```
pair<int, int> p = {10, 20};

cout << p.first << '\n';   // 10
cout << p.second << '\n';  // 20

p.first = 100;
p.second = 200;

// p = {100, 200}
```

A pair can also be returned from a function:

```
pair<int, int> getMinMax(vector<int>& a) {
    int mn = *min_element(a.begin(), a.end());
    int mx = *max_element(a.begin(), a.end());

    return {mn, mx};
}
```

### C. Pair comparison operators

Pairs support `==`, `!=`, `<`, `>`, `<=`, and `>=` when their element types support the corresponding comparisons.

```
pair<int, int> a = {1, 5};
pair<int, int> b = {1, 8};
pair<int, int> c = {2, 1};

cout << (a < b);  // 1
cout << (b < c);  // 1
```

Why?

- `{1,5} < {1,8}` because the first elements are equal and `5 < 8`.
    
- `{1,8} < {2,1}` because `1 < 2` for the first elements.
    

This ordering is called lexicographical ordering.

### D. Sorting pairs

By default, `sort()` sorts pairs in ascending lexicographical order.

```
vector<pair<int, int>> v = {
    {3, 10},
    {1, 20},
    {3, 5},
    {2, 15}
};

sort(v.begin(), v.end());

for (auto [x, y] : v) {
    cout << x << " " << y << '\n';
}
```

Output:

```
1 20
2 15
3 5
3 10
```

The structured binding `auto [x, y]` requires C++17.

Sort by first ascending, then second descending:

```
sort(v.begin(), v.end(), [](auto a, auto b) {
    if (a.first != b.first)
        return a.first < b.first;

    return a.second > b.second;
});
```

Sort by second element ascending:

```
sort(v.begin(), v.end(), [](auto a, auto b) {
    return a.second < b.second;
});
```

These custom comparators are especially useful in Codeforces and other contest problems.

### E. Pair with vectors

A vector of pairs stores multiple two-element records.

```
vector<pair<int, int>> v;

v.push_back({10, 100});
v.push_back({20, 200});
v.emplace_back(30, 300);
```

Access elements using:

```
cout << v[0].first;
cout << v[0].second;
```

Loop through the vector:

```
for (const auto& p : v) {
    cout << p.first << " " << p.second << '\n';
}
```

Complexity: Accessing a vector element by index takes \(O(1)\); sorting \(n\) pairs takes \(O(n\log n)\).

### F. Pair in priority queues

Pairs are frequently used in graph algorithms.

```
priority_queue<
    pair<int, int>,
    vector<pair<int, int>>,
    greater<pair<int, int>>
> pq;

pq.push({10, 2});
pq.push({5, 1});
pq.push({5, 3});

while (!pq.empty()) {
    auto [distance, node] = pq.top();
    pq.pop();

    cout << distance << " " << node << '\n';
}
```

Output:

```
5 1
5 3
10 2
```

`greater<pair<int,int>>` creates a min-priority queue. It orders by the first element, then by the second when the first elements tie.

This is useful for Dijkstra's algorithm, where the pair represents `{distance, node}`.

### G. Pair in maps and sets

Pairs can be used as keys in ordered associative containers.

```
map<pair<int, int>, int> mp;

mp[{1, 2}] = 100;
mp[{3, 4}] = 200;

cout << mp[{1, 2}];  // 100
```

A set of pairs automatically maintains sorted order and removes duplicate pairs.

```
set<pair<int, int>> s;

s.insert({2, 3});
s.insert({1, 5});
s.insert({2, 3});  // Duplicate

// Contains {1,5} and {2,3}
```

### H. Unpacking pairs

Using structured bindings (C++17+):

```
pair<int, int> p = {10, 20};

auto [x, y] = p;

cout << x << " " << y;
```

This creates copies of the two elements. To bind by reference and modify the original pair:

```
auto& [x, y] = p;
x = 100;
```

Using `tie()` (C++11+):

```
int x, y;
pair<int, int> p = {10, 20};

tie(x, y) = p;
```

### I. Nested pairs

A pair can contain another pair.

```
pair<int, pair<int, int>> p = {1, {2, 3}};

cout << p.first;         // 1
cout << p.second.first;  // 2
cout << p.second.second; // 3
```

Nested pairs can be useful, but for complex data, a `struct` may improve readability.

### J. Important pitfalls

1. Overflow: Use `long long` for large integers.
    
    ```
    pair<long long, int> p;
    ```
    
2. Incorrect sorting assumptions: Default sorting uses both elements, not just `.first`.
    
3. Uninitialized built-in values: A default-initialized local `pair<int,int>` does not guarantee zero values. Use `pair<int,int> p{};` to initialize both to zero.
    
4. Pair versus tuple: A pair stores two elements; a `tuple` can store any fixed number of elements.
    
5. Pair versus struct: Use a pair for simple two-value relationships. Use a struct when the fields need meaningful names or the record has more complex behavior.
    

## 3. Competitive programming patterns

Pattern 1 — Value with original index

```
vector<pair<int, int>> v;

for (int i = 0; i < n; i++)
    v.push_back({a[i], i});

sort(v.begin(), v.end());
```

Useful when sorting values while preserving their original positions.

Pattern 2 — Coordinates

```
pair<int, int> cell = {row, col};
```

Useful in grid traversal and BFS.

Pattern 3 — Adjacency list

```
vector<vector<pair<int, int>>> adj(n);

adj[u].push_back({v, weight});
```

Stores the neighbor and edge weight for weighted graphs.

Pattern 4 — Interval sorting

```
vector<pair<int, int>> intervals;
sort(intervals.begin(), intervals.end());
```

Sorts intervals by start time, then by end time.

## 4. Quick complexity reference

|Operation|Complexity|
|---|---|
|Access `.first` / `.second`|\(O(1)\)|
|Assign a pair|Usually \(O(1)\) for primitive types|
|Compare two integer pairs|\(O(1)\)|
|Sort a vector of \(n\) pairs|\(O(n\log n)\)|
|Insert into `set<pair<...>>`|\(O(\log n)\)|
|Insert into `map<pair<...>, ...>`|\(O(\log n)\)|
|Push/pop in a priority queue|\(O(\log n)\)|

Final takeaway: Master `pair`, lexicographical sorting, custom comparators, and `priority_queue<pair<...>>`. These are among the most frequently useful STL techniques in competitive programming.

