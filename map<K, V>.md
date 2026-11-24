
# C++ Competitive Programming: `map<K, V>`

## 1. Short notes (quick revision)

### Definition

A `map<K, V>` stores key-value pairs in sorted order of keys. Each key is unique, and inserting an existing key updates its mapped value.

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    map<int, string> mp;

    mp[1] = "Alice";
    mp[2] = "Bob";
    mp[3] = "Charlie";

    cout << mp[2];  // Bob
}
```

### Important operations

|Operation|Purpose|Complexity|
|---|---|---|
|`mp[key] = value`|Insert or update|\(O(\log n)\)|
|`mp.insert({key, value})`|Insert if key is absent|\(O(\log n)\)|
|`mp.emplace(key, value)`|Construct and insert|\(O(\log n)\)|
|`mp.find(key)`|Find a key|\(O(\log n)\)|
|`mp.count(key)`|Check key existence (0 or 1)|\(O(\log n)\)|
|`mp.contains(key)`|Check existence (C++20)|\(O(\log n)\)|
|`mp.erase(key)`|Remove by key|\(O(\log n)\)|
|`mp.size()`|Number of entries|\(O(1)\)|
|`mp.empty()`|Check whether empty|\(O(1)\)|
|`mp.lower_bound(key)`|First key not less than key|\(O(\log n)\)|
|`mp.upper_bound(key)`|First key greater than key|\(O(\log n)\)|

Here, \(n\) is the number of entries.

### Iterating through a map

```
for (auto [key, value] : mp) {
    cout << key << " " << value << '\n';
}
```

Keys are visited in ascending order by default. Structured bindings require C++17.

### Common competitive programming uses

- Frequency counting
    
- Key-value lookup
    
- Coordinate compression
    
- Maintaining ordered data
    
- Counting occurrences of strings
    
- Prefix/range queries using ordered keys
    

Remember: `map` stores unique keys in sorted order. For average \(O(1)\) lookup without sorted ordering, consider `unordered_map`.

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
map<int, int> mp;
map<string, int> frequency;
map<int, string> students;

map<int, int> a = {
    {1, 100},
    {2, 200},
    {3, 300}
};
```

The first template parameter is the key type, and the second is the mapped-value type.

```
map<string, vector<int>> groups;
map<pair<int, int>, int> coordinates;
```

A `map` is typically implemented as a balanced binary search tree, commonly a red-black tree, which maintains sorted keys.

### B. Inserting and updating elements

Using `operator[]`:

```
map<int, string> mp;

mp[1] = "Apple";
mp[2] = "Banana";

mp[1] = "Orange";  // Updates existing key 1
```

If the key does not exist, `mp[key]` inserts it and value-initializes the mapped value before assignment.

```
map<int, int> mp;

mp[5]++;  // Creates key 5 with value 0, then increments to 1
```

This makes `map` especially convenient for frequency counting.

Using `insert()`:

```
mp.insert({3, "Grapes"});
```

`insert()` does not overwrite an existing key.

Using `emplace()`:

```
map<int, string> mp;
mp.emplace(1, "Alice");
```

`emplace()` constructs an element in the container if insertion succeeds.

### C. Accessing elements safely

```
map<int, string> mp;
mp[1] = "Alice";

cout << mp[1];  // Alice
```

To access a value without inserting a missing key, use `find()`:

```
auto it = mp.find(2);

if (it != mp.end()) {
    cout << it->second;
} else {
    cout << "Key not found";
}
```

Important: `mp[2]` would insert key `2` if it did not exist. `find()` does not modify the map.

### D. Checking whether a key exists

```
if (mp.count(10)) {
    cout << "Exists";
}
```

In C++20 and later:

```
if (mp.contains(10)) {
    cout << "Exists";
}
```

Because map keys are unique, `count(key)` can only return `0` or `1`.

### E. Erasing elements

```
map<int, int> mp = {
    {1, 10},
    {2, 20},
    {3, 30}
};

mp.erase(2);  // Erase by key
```

Erase using an iterator:

```
auto it = mp.find(3);

if (it != mp.end())
    mp.erase(it);
```

Erase a range:

```
mp.erase(mp.begin(), mp.end());  // Erase all entries
```

`mp.clear()` is another way to remove all entries.

### F. Iterating through a map

Range-based loop:

```
for (const auto& [key, value] : mp) {
    cout << key << " " << value << '\n';
}
```

Iterator-based loop:

```
for (auto it = mp.begin(); it != mp.end(); ++it) {
    cout << it->first << " " << it->second << '\n';
}
```

- `it->first` is the key.
    
- `it->second` is the mapped value.
    
- Iteration follows the map's key ordering.
    

Map keys cannot be modified through an iterator because changing a key in place could violate the ordering invariant. Mapped values can be modified.

### G. Frequency counting

This is one of the most common competitive programming applications.

```
vector<int> a = {1, 2, 1, 3, 2, 1};

map<int, int> freq;

for (int x : a) {
    freq[x]++;
}

for (const auto& [x, count] : freq) {
    cout << x << " -> " << count << '\n';
}
```

Output:

```
1 -> 3
2 -> 2
3 -> 1
```

Complexity: \(O(n\log n)\) time and \(O(k)\) space, where \(k\) is the number of distinct values.

For counting characters or strings, the same pattern works:

```
map<string, int> freq;

for (string s : {"cat", "dog", "cat"}) {
    freq[s]++;
}
```

### H. `lower_bound()` and `upper_bound()`

These operations are useful when finding positions in a sorted map.

```
map<int, string> mp = {
    {10, "A"},
    {20, "B"},
    {30, "C"},
    {40, "D"}
};

auto it1 = mp.lower_bound(25);  // Key 30
auto it2 = mp.upper_bound(30);  // Key 40
```

- `lower_bound(x)`: first key greater than or equal to `x`.
    
- `upper_bound(x)`: first key strictly greater than `x`.
    

Both return `mp.end()` if no matching key exists.

Count keys in the half-open interval \([L,R)\):

```
int cnt = distance(
    mp.lower_bound(L),
    mp.lower_bound(R)
);
```

Note that `std::distance` on map iterators takes linear time in the number of entries traversed, even though each bound lookup is \(O(\log n)\). If you need frequent range-count queries, a different data structure may be more suitable.

### I. Custom ordering

By default, `map` sorts keys in ascending order.

For descending order:

```
map<int, string, greater<int>> mp;

mp[1] = "A";
mp[3] = "C";
mp[2] = "B";
```

The iteration order is `3, 2, 1`.

For a custom key type, define a comparator that provides a strict weak ordering:

```
struct Compare {
    bool operator()(int a, int b) const {
        return a > b;
    }
};

map<int, string, Compare> mp;
```

You cannot directly specify a comparator that sorts map entries by their mapped values. If you need that ordering, copy the entries into a vector and sort them with a custom comparator.

### J. `map` vs. `unordered_map`

|Feature|`map`|`unordered_map`|
|---|---|---|
|Key ordering|Sorted|Unspecified|
|Typical lookup|\(O(\log n)\)|Average \(O(1)\)|
|Worst-case lookup|\(O(\log n)\)|\(O(n)\)|
|Implementation|Balanced tree|Hash table|
|`lower_bound()`|Supported|Not supported|
|Custom ordering|Supported|Not applicable|
|Useful for|Ordered operations|Fast average lookup|

Use `map` when you need sorted keys, predecessor/successor searches, or predictable logarithmic operations. Use `unordered_map` when key ordering is unnecessary and average constant-time lookup is useful.

### K. Coordinate compression

Coordinate compression maps large or sparse values to small consecutive indices while preserving their relative order.

```
vector<int> a = {1000000000, 50, 1000000000, 7};

vector<int> vals = a;

sort(vals.begin(), vals.end());
vals.erase(unique(vals.begin(), vals.end()), vals.end());

map<int, int> id;

for (int i = 0; i < (int)vals.size(); i++) {
    id[vals[i]] = i;
}

// id[7]          = 0
// id[50]         = 1
// id[1000000000] = 2
```

This technique is useful when values are large but the number of distinct values is manageable. Since the sorted unique vector is already available, you can also use `lower_bound()` instead of a map for compression lookups.

## 3. Complexity reference

|Operation|Complexity|
|---|---|
|Insert or update|\(O(\log n)\)|
|Find a key|\(O(\log n)\)|
|Erase by key|\(O(\log n)\)|
|`lower_bound()` / `upper_bound()`|\(O(\log n)\)|
|`size()` / `empty()`|\(O(1)\)|
|Traverse all entries|\(O(n)\)|
|Count frequencies of \(n\) items|\(O(n\log n)\)|

## 4. Competitive programming patterns

Pattern 1 — Count occurrences

```
map<int, int> freq;
for (int x : a) freq[x]++;
```

Maintains counts in sorted key order.

Pattern 2 — Find a key safely

```
auto it = mp.find(key);
if (it != mp.end())
    cout << it->second;
```

Avoids accidentally inserting missing keys.

Pattern 3 — Find the next available key

```
auto it = mp.upper_bound(x);
if (it != mp.end())
    cout << it->first;
```

Finds the smallest key strictly greater than `x`.

Pattern 4 — Store pair keys

```
map<pair<int, int>, int> mp;
mp[{2, 3}] = 10;
```

Useful for grid coordinates, ordered pairs, and counting coordinate occurrences.

Final takeaway: Master `map` insertion, lookup, frequency counting, ordered iteration, `lower_bound()`, and `upper_bound()`. These operations are especially valuable in problems that require both key-value storage and sorted order.