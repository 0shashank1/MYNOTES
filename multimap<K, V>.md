
# C++ Competitive Programming: `multimap<K, V>`

## 1. Short notes (quick revision)

### Definition

A `multimap<K, V>` is an STL associative container that stores key-value pairs in sorted key order and allows duplicate keys. Unlike `map`, multiple elements can have the same key.

cppreference.com

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    multimap<int, string> mp;

    mp.insert({1, "Alice"});
    mp.insert({1, "Bob"});
    mp.insert({2, "Charlie"});

    for (auto [key, value] : mp) {
        cout << key << " " << value << '\n';
    }
}
```

Output:

```
1 Alice
1 Bob
2 Charlie
```

### Important operations

|Operation|Purpose|Complexity|
|---|---|---|
|`mp.insert({k, v})`|Insert a key-value pair|\(O(\log n)\)|
|`mp.emplace(k, v)`|Construct and insert an entry|\(O(\log n)\)|
|`mp.find(k)`|Find one matching key|\(O(\log n)\)|
|`mp.count(k)`|Count entries with key `k`|\(O(\log n + c)\)|
|`mp.contains(k)`|Check existence (C++20)|\(O(\log n)\)|
|`mp.erase(k)`|Remove all entries with key `k`|\(O(\log n+c)\)|
|`mp.erase(it)`|Remove one entry|Amortized \(O(1)\)|
|`mp.lower_bound(k)`|First key not less than `k`|\(O(\log n)\)|
|`mp.upper_bound(k)`|First key greater than `k`|\(O(\log n)\)|
|`mp.equal_range(k)`|Get the range of matching keys|\(O(\log n)\)|
|`mp.size()`|Total number of entries|\(O(1)\)|
|`mp.empty()`|Check if empty|\(O(1)\)|

Here, \(n\) is the number of entries and \(c\) is the number of entries matching the key.

Remember these three rules:

- `multimap` allows duplicate keys.
    
- `multimap` does not provide `operator[]` or `at()`.
    
- `erase(key)` removes all entries with that key; `erase(iterator)` removes only one.
    

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
multimap<int, int> mp;
multimap<string, int> scores;
multimap<int, string> students;

multimap<int, string> mp2 = {
    {1, "Alice"},
    {1, "Bob"},
    {2, "Charlie"}
};
```

The first template parameter is the key type; the second is the mapped-value type.

By default, keys are sorted in ascending order. Entries with equivalent keys retain their insertion order in modern C++ (C++11 and later).

cppreference.com

### B. Inserting elements

Use `insert()` or `emplace()` to add entries.

```
multimap<int, string> mp;

mp.insert({10, "A"});
mp.insert({10, "B"});
mp.insert({20, "C"});

mp.emplace(10, "D");
```

All four entries are retained, including the three entries with key `10`.

Important difference from `map`: inserting an existing key into a `multimap` adds another entry instead of replacing the existing value.

### C. Accessing and iterating through elements

A `multimap` does not support direct access using `mp[key]`, because a key can correspond to multiple values.

Iterate through every entry:

```
for (auto it = mp.begin(); it != mp.end(); ++it) {
    cout << it->first << " "
         << it->second << '\n';
}
```

Or use a range-based loop:

```
for (const auto& [key, value] : mp) {
    cout << key << " " << value << '\n';
}
```

The second example uses structured bindings (C++17).

### D. Finding a key

The `find()` function returns an iterator to one matching entry, or `mp.end()` if the key does not exist.

```
auto it = mp.find(10);

if (it != mp.end()) {
    cout << it->first << " "
         << it->second;
}
```

When multiple entries have the same key, `find()` does not return all of them. Use `equal_range()` to process every matching entry.

### E. Finding all values for a key using `equal_range()`

This is one of the most important operations for competitive programming.

```
multimap<int, string> mp;

mp.insert({1, "Alice"});
mp.insert({1, "Bob"});
mp.insert({1, "Charlie"});
mp.insert({2, "David"});

auto [first, last] = mp.equal_range(1);

for (auto it = first; it != last; ++it) {
    cout << it->second << '\n';
}
```

Output:

```
Alice
Bob
Charlie
```

`equal_range(key)` returns two iterators:

- `first`: the first entry whose key is equivalent to the requested key.
    
- `last`: the iterator immediately after the last matching entry.
    

This lets you process all values for one key without scanning the entire container.

### F. Counting duplicate keys

```
multimap<int, string> mp;

mp.insert({5, "A"});
mp.insert({5, "B"});
mp.insert({5, "C"});
mp.insert({8, "D"});

cout << mp.count(5);  // 3
cout << mp.count(8);  // 1
cout << mp.count(9);  // 0
```

For a `multimap`, `count(key)` can return any non-negative count up to the number of entries in the container. Its complexity is \(O(\log n+c)\), where \(c\) is the number of matching entries.

### G. Deleting elements

Remove all entries with a given key:

```
mp.erase(5);
```

Remove just one matching entry:

```
auto it = mp.find(5);

if (it != mp.end()) {
    mp.erase(it);
}
```

Remove every entry:

```
mp.clear();
```

This distinction is important: `erase(key)` deletes every entry with that key, whereas `erase(iterator)` removes only the selected entry.

### H. `lower_bound()` and `upper_bound()`

These functions help you search the ordered keys.

```
multimap<int, string> mp = {
    {10, "A"},
    {20, "B"},
    {20, "C"},
    {30, "D"}
};

auto it1 = mp.lower_bound(20); // First key >= 20
auto it2 = mp.upper_bound(20); // First key > 20
```

For this example:

- `lower_bound(20)` points to the first entry with key `20`.
    
- `upper_bound(20)` points to the entry with key `30`.
    

If there is no qualifying entry, the returned iterator is `mp.end()`.

### I. Descending key order

You can use `greater<int>` to order keys in descending order.

```
multimap<int, string, greater<int>> mp;

mp.insert({10, "A"});
mp.insert({30, "B"});
mp.insert({20, "C"});

for (const auto& [key, value] : mp) {
    cout << key << " " << value << '\n';
}
```

Output:

```
30 B
20 C
10 A
```

The comparator changes the key ordering, not the ordering based on mapped values.

### J. Frequency counting: `map` vs. `multimap`

A common question is whether `multimap` should be used for frequency counting.

Using `map` — usually better for frequencies:

```
vector<int> a = {1, 2, 1, 3, 2, 1};
map<int, int> freq;

for (int x : a)
    freq[x]++;
```

Result: `1 → 3`, `2 → 2`, `3 → 1`.

Using `multimap` — retains each individual occurrence:

```
multimap<int, int> mp;

mp.insert({1, 100});
mp.insert({1, 200});
mp.insert({1, 300});
```

Here, key `1` has three separate mapped values.

Use `map<int,int>` when you want one count per distinct number. Use `multimap<int,int>` when you need to preserve multiple separate records under the same key.

## 3. `map` vs. `multimap` vs. `unordered_multimap`

|Feature|`map`|`multimap`|`unordered_multimap`|
|---|---|---|---|
|Duplicate keys|No|Yes|Yes|
|Sorted keys|Yes|Yes|No|
|`operator[]`|Yes|No|No|
|Typical lookup|\(O(\log n)\)|\(O(\log n)\)|Average \(O(1)\)|
|`equal_range()`|Yes|Yes|Yes|
|`lower_bound()`|Yes|Yes|No|
|Best use|Unique ordered keys|Multiple ordered records per key|Fast average lookup with duplicates|

## 4. Competitive programming patterns

Pattern 1 — Store multiple values for a key

```
multimap<int, string> mp;
mp.insert({1, "Apple"});
mp.insert({1, "Banana"});
```

Useful when one identifier is associated with several records.

Pattern 2 — Process all entries for one key

```
auto [l, r] = mp.equal_range(key);

for (auto it = l; it != r; ++it)
    cout << it->second << '\n';
```

Avoids scanning unrelated keys.

Pattern 3 — Maintain duplicate values in sorted order

```
multimap<int, int> events;
events.insert({5, 101});
events.insert({5, 102});
events.insert({8, 103});
```

Useful for event times, grouped records, and schedules where timestamps can repeat.

## 5. Final revision checklist

- `multimap` stores key-value pairs and permits duplicate keys.
    
- Entries are sorted by key by default.
    
- Use `insert()` or `emplace()` to add elements.
    
- Use `find()` to locate one matching entry.
    
- Use `equal_range()` to iterate over all entries with a key.
    
- `erase(key)` removes all matching entries.
    
- `multimap` does not support `operator[]`.
    
- Search, insertion, and removal are generally logarithmic, plus any work required to process duplicates.
    

For the official reference, see [cppreference — `std::multimap`](https://en.cppreference.com/w/cpp/container/multimap?utm_source=chatgpt.com).