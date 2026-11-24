
# C++ Competitive Programming: `unordered_map<K, V>`

## 1. Short notes (quick revision)

### Definition

An `unordered_map<K, V>` stores unique key-value pairs using a hash table. Unlike `map`, it does not maintain keys in sorted order. It typically provides average \(O(1)\) insertion, lookup, and deletion.

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    unordered_map<int, string> mp;

    mp[1] = "Alice";
    mp[2] = "Bob";
    mp[3] = "Charlie";

    cout << mp[2];  // Bob
}
```

### Important operations

|Operation|Purpose|Average complexity|
|---|---|---|
|`mp[key] = value`|Insert or update|\(O(1)\)|
|`mp.insert({key, value})`|Insert if absent|\(O(1)\)|
|`mp.emplace(key, value)`|Construct and insert|\(O(1)\)|
|`mp.find(key)`|Find a key|\(O(1)\)|
|`mp.count(key)`|Check existence (0 or 1)|\(O(1)\)|
|`mp.contains(key)`|Check existence (C++20)|\(O(1)\)|
|`mp.erase(key)`|Remove a key|\(O(1)\)|
|`mp.size()`|Number of entries|\(O(1)\)|
|`mp.empty()`|Check whether empty|\(O(1)\)|
|`mp.reserve(n)`|Reserve bucket capacity for elements|Average \(O(n)\) when rehashing is needed|

Important: These are average-case complexities. In the worst case, hash collisions can make lookup, insertion, and deletion \(O(n)\).

### Common competitive programming uses

- Frequency counting
    
- Two Sum and pair-sum problems
    
- Fast key-value lookup
    
- Counting strings and characters
    
- Tracking visited states
    
- Memoization and dynamic programming
    

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
unordered_map<int, int> mp;
unordered_map<string, int> freq;
unordered_map<int, string> names;

unordered_map<int, int> scores = {
    {1, 90},
    {2, 85},
    {3, 95}
};
```

The first template parameter specifies the key type, and the second specifies the mapped-value type.

Common declarations:

```
unordered_map<long long, int> mp;
unordered_map<string, vector<int>> groups;
unordered_map<int, bool> visited;
```

Keys must be hashable. Standard types such as integers and strings have standard hash support.

### B. Insertion and updating

Using `operator[]`:

```
unordered_map<int, string> mp;

mp[1] = "Alice";
mp[2] = "Bob";

mp[1] = "Charlie";  // Updates the existing value
```

If a key does not exist, `mp[key]` inserts it and value-initializes its mapped value before assignment.

```
unordered_map<int, int> freq;

freq[10]++;  // Creates key 10 with value 0, then increments to 1
```

Using `insert()`:

```
mp.insert({3, "David"});
```

If key `3` already exists, its value is not overwritten.

Using `emplace()`:

```
mp.emplace(4, "Emma");
```

Constructs and inserts an entry if the key is not already present.

### C. Accessing values safely

```
unordered_map<int, string> mp;
mp[1] = "Alice";

auto it = mp.find(2);

if (it != mp.end()) {
    cout << it->second;
} else {
    cout << "Key not found";
}
```

Why use `find()` instead of `mp[2]`?

- `find()` does not insert a missing key.
    
- `mp[2]` inserts key `2` if it is absent.
    

Use `find()` when you only want to check or retrieve an existing entry.

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

Since keys are unique, `count(key)` returns either `0` or `1`.

### E. Iterating through an unordered map

```
unordered_map<int, string> mp = {
    {1, "Alice"},
    {2, "Bob"},
    {3, "Charlie"}
};

for (const auto& [key, value] : mp) {
    cout << key << " " << value << '\n';
}
```

This uses C++17 structured bindings.

Important: The iteration order is unspecified. Never assume entries will appear in ascending order or in insertion order.

### F. Frequency counting — the most common pattern

```
vector<int> a = {1, 2, 1, 3, 2, 1};

unordered_map<int, int> freq;

for (int x : a) {
    freq[x]++;
}

for (const auto& [x, count] : freq) {
    cout << x << " -> " << count << '\n';
}
```

The counts are:

- `1 → 3`
    
- `2 → 2`
    
- `3 → 1`
    

The output order is unspecified.

Complexity:

- Average time: \(O(n)\)
    
- Space: \(O(k)\), where \(k\) is the number of distinct keys.
    

This is generally faster than `map` for frequency counting when sorted order is unnecessary.

### G. Two Sum problem using `unordered_map`

Given an array and a target, find two distinct indices whose values sum to the target.

```
vector<int> twoSum(vector<int>& a, int target) {
    unordered_map<int, int> mp;

    for (int i = 0; i < (int)a.size(); i++) {
        int need = target - a[i];

        auto it = mp.find(need);

        if (it != mp.end()) {
            return {it->second, i};
        }

        mp[a[i]] = i;
    }

    return {};
}
```

Example:

```
Array:  [2, 7, 11, 15]
Target: 9
Answer: [0, 1]
```

The map stores previously seen values and their indices. Checking for the complement before inserting the current element ensures that an index is not paired with itself.

Complexity: Average \(O(n)\) time and \(O(n)\) space.

### H. Erasing elements

Erase by key:

```
mp.erase(10);
```

Erase using an iterator:

```
auto it = mp.find(20);

if (it != mp.end()) {
    mp.erase(it);
}
```

Clear the entire container:

```
mp.clear();
```

Erasing by key or iterator is average \(O(1)\), but worst-case \(O(n)\).

### I. `reserve()` and `load_factor()`

For large inputs, reserving space can reduce repeated rehashing.

```
unordered_map<int, int> mp;

mp.reserve(100000);

for (int i = 0; i < 100000; i++) {
    mp[i] = i * 2;
}
```

Useful related functions:

```
mp.bucket_count();  // Number of buckets
mp.load_factor();   // Elements per bucket on average
mp.max_load_factor(); // Maximum load-factor threshold
```

You can set the load-factor threshold before reserving:

```
mp.max_load_factor(0.7);
mp.reserve(100000);
```

A lower load factor may reduce collisions but increase memory usage.

### J. Custom key types

For a custom key such as `pair<int,int>`, the standard library does not provide a universal `std::hash<pair<int,int>>` in ordinary C++17 implementations. You can supply your own hash function.

```
struct PairHash {
    size_t operator()(const pair<int, int>& p) const {
        size_t h1 = hash<int>{}(p.first);
        size_t h2 = hash<int>{}(p.second);

        return h1 ^ (h2 + 0x9e3779b9
                     + (h1 << 6) + (h1 >> 2));
    }
};

unordered_map<pair<int, int>, int, PairHash> mp;

mp[{1, 2}] = 10;
mp[{3, 4}] = 20;
```

This is useful for coordinate counting, grid states, and memoization. Hash collisions are possible, but `unordered_map` resolves them internally using key equality.

### K. `map` vs. `unordered_map`

|Feature|`map`|`unordered_map`|
|---|---|---|
|Key ordering|Sorted|Unspecified|
|Average lookup|\(O(\log n)\)|\(O(1)\)|
|Worst-case lookup|\(O(\log n)\)|\(O(n)\)|
|Implementation|Balanced tree|Hash table|
|`lower_bound()`|Supported|Not supported|
|Indexing with `[]`|Supported|Supported|
|Custom key ordering|Supported|No ordering comparator|
|Best use|Ordered queries|Fast average lookup|

Competitive programming rule: Prefer `unordered_map` for frequency counting and fast lookup when order is irrelevant. Prefer `map` when you need ordered keys, `lower_bound()`, or predictable logarithmic worst-case performance.

## 3. Complexity reference

|Operation|Average|Worst case|
|---|---|---|
|Insert|\(O(1)\)|\(O(n)\)|
|Find|\(O(1)\)|\(O(n)\)|
|Erase by key|\(O(1)\)|\(O(n)\)|
|Check existence|\(O(1)\)|\(O(n)\)|
|Traverse all entries|\(O(n)\)|\(O(n)\)|
|Count frequencies of \(n\) items|\(O(n)\)|\(O(n^2)\)|

Worst-case behavior matters when inputs may be adversarial or hashing performance is unpredictable.

## 4. Competitive programming patterns

Pattern 1 — Frequency counting

```
unordered_map<int, int> freq;
for (int x : a)
    freq[x]++;
```

Counts each distinct value with average constant-time updates.

Pattern 2 — Safe lookup

```
auto it = mp.find(key);
if (it != mp.end())
    cout << it->second;
```

Checks for a key without accidentally inserting it.

Pattern 3 — Fast membership testing

```
unordered_map<int, bool> seen;
seen[x] = true;

if (seen.count(y))
    cout << "Found";
```

For membership alone, `unordered_set<int>` is usually more appropriate than `unordered_map<int,bool>`.

Pattern 4 — Prefix-sum frequency

```
unordered_map<long long, int> freq;
long long pref = 0;

freq[0] = 1;

for (int x : a) {
    pref += x;
    // Count subarrays with sum k:
    ans += freq[pref - k];
    freq[pref]++;
}
```

A standard average \(O(n)\) solution for counting subarrays with sum `k`. Declare `ans` as `long long` when the answer may be large.

## 5. Final revision checklist

- `unordered_map` stores unique keys using hashing.
    
- Average insertion, lookup, and deletion are \(O(1)\).
    
- Worst-case lookup, insertion, and deletion can be \(O(n)\).
    
- `operator[]` inserts a missing key; `find()` does not.
    
- Iteration order is unspecified.
    
- `reserve()` can reduce rehashing.
    
- Use a custom hash for unsupported key types such as `pair<int,int>`.
    
- Choose `map` when sorted order is required and `unordered_map` when average constant-time lookup is the priority.
    

One-line revision: `unordered_map<K,V>` is the go-to STL container for fast average-case key-value lookup, frequency counting, and hashing-based competitive programming problems.