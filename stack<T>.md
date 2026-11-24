

# C++ Competitive Programming: `stack<T>`

## 1. Short notes (quick revision)

### Definition

A `stack<T>` is an STL container adapter that follows LIFO (Last In, First Out). The last element inserted is the first element removed.

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    stack<int> st;

    st.push(10);
    st.push(20);
    st.push(30);

    cout << st.top();  // 30

    st.pop();          // Removes 30
    cout << st.top();  // 20
}
```

### Important operations

|Operation|Purpose|Complexity|
|---|---|---|
|`st.push(x)`|Insert an element at the top|\(O(1)\)|
|`st.emplace(x)`|Construct an element at the top|\(O(1)\)|
|`st.pop()`|Remove the top element|\(O(1)\)|
|`st.top()`|Access the top element|\(O(1)\)|
|`st.size()`|Get the number of elements|\(O(1)\)|
|`st.empty()`|Check whether empty|\(O(1)\)|
|`st.swap(other)`|Swap two stacks|Usually \(O(1)\)|

Remember: `pop()` removes the top element but does not return it.

```
int x = st.top();
st.pop();
```

### Common competitive programming uses

- Balanced parentheses
    
- Next greater element
    
- Next smaller element
    
- Monotonic stack problems
    
- Expression evaluation
    
- Iterative DFS
    
- Undo operations and backtracking
    

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
stack<int> st;
stack<char> chars;
stack<string> words;
stack<pair<int, int>> points;
stack<long long> values;
```

The default underlying container is `deque`.

You can also specify another compatible container:

```
stack<int, vector<int>> st;
```

### B. Insertion, access, and removal

```
stack<int> st;

st.push(10);
st.push(20);
st.push(30);

// Bottom -> 10, 20, 30 <- Top

cout << st.top();  // 30

st.pop();

// Bottom -> 10, 20 <- Top
cout << st.top();  // 20
```

To process every element from top to bottom:

```
while (!st.empty()) {
    cout << st.top() << " ";
    st.pop();
}
```

Output for the original stack:

```
30 20 10
```

Note that this loop destroys the contents of the stack.

### C. Checking size and emptiness

```
stack<int> st;

cout << st.empty();  // 1 (true)

st.push(5);
st.push(10);

cout << st.size();   // 2
cout << st.empty();  // 0 (false)
```

Never call `top()` or `pop()` on an empty stack; doing so results in undefined behavior.

### D. Balanced parentheses

A classic stack problem is checking whether brackets are balanced.

```
bool isBalanced(string s) {
    stack<char> st;

    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') {
            st.push(c);
        } else {
            if (st.empty())
                return false;

            char t = st.top();
            st.pop();

            if ((c == ')' && t != '(') ||
                (c == ']' && t != '[') ||
                (c == '}' && t != '{')) {
                return false;
            }
        }
    }

    return st.empty();
}
```

Examples:

- `"{[()]}"` → `true`
    
- `"([)]"` → `false`
    
- `"((("` → `false`
    

This implementation assumes the input contains only bracket characters. If arbitrary characters are allowed, explicitly handle them.

Complexity: \(O(n)\) time and \(O(n)\) space.

### E. Next greater element

Given an array, find the first greater element to the right of each element. A monotonic stack solves this efficiently.

```
vector<int> nextGreater(vector<int>& a) {
    int n = a.size();
    vector<int> ans(n, -1);
    stack<int> st;  // Stores indices

    for (int i = 0; i < n; i++) {
        while (!st.empty() &&
               a[st.top()] < a[i]) {
            ans[st.top()] = a[i];
            st.pop();
        }

        st.push(i);
    }

    return ans;
}
```

Example:

```
Array:  [4, 5, 2, 10]
Answer: [5, 10, 10, -1]
```

The stack stores indices whose next greater elements have not yet been found.

Complexity: \(O(n)\) time and \(O(n)\) space. Each index is pushed and popped at most once.

### F. Iterative depth-first search (DFS)

A stack can replace recursion for graph traversal.

```
vector<bool> visited(n, false);
stack<int> st;

st.push(src);

while (!st.empty()) {
    int u = st.top();
    st.pop();

    if (visited[u])
        continue;

    visited[u] = true;
    cout << u << " ";

    for (int v : adj[u]) {
        if (!visited[v])
            st.push(v);
    }
}
```

This version may push the same vertex multiple times, but the visited check prevents processing it more than once. The exact traversal order depends on adjacency-list order.

### G. Reversing a string using a stack

```
string reverseString(string s) {
    stack<char> st;

    for (char c : s)
        st.push(c);

    string ans;

    while (!st.empty()) {
        ans += st.top();
        st.pop();
    }

    return ans;
}
```

Example: `"hello"` becomes `"olleh"`.

Complexity: \(O(n)\) time and \(O(n)\) auxiliary space.

### H. Copying and swapping stacks

Stacks can be copied directly:

```
stack<int> a;
a.push(1);
a.push(2);

stack<int> b = a;
```

Now `a` and `b` contain the same elements independently.

Swap two stacks:

```
stack<int> a, b;

a.push(10);
b.push(20);

a.swap(b);
```

After swapping, `a.top()` is `20` and `b.top()` is `10`.

There is no standard `clear()` member for `stack`; remove elements in a loop if you need to empty it.

## 3. Complexity reference

|Operation or algorithm|Complexity|
|---|---|
|`push`, `pop`, `top`|\(O(1)\)|
|`size`, `empty`|\(O(1)\)|
|Balanced parentheses|\(O(n)\)|
|Reverse a string|\(O(n)\)|
|Next greater element|\(O(n)\)|
|Iterative DFS|\(O(V+E)\) with adjacency lists, accounting for graph traversal|

## 4. Queue vs. stack

|Feature|`stack`|`queue`|
|---|---|---|
|Principle|LIFO|FIFO|
|Access|`top()`|`front()`, `back()`|
|Insert|`push()`|`push()`|
|Remove|`pop()`|`pop()`|
|Typical application|DFS, parentheses|BFS, level-order traversal|

## 5. Competitive programming tips

1. Use `stack<int>` for values or indices.
    
2. Use `stack<pair<int,int>>` when each entry needs two related values.
    
3. Learn monotonic stacks; they often reduce \(O(n^2)\) solutions to \(O(n)\).
    
4. Check `!st.empty()` before accessing `st.top()`.
    
5. Remember that standard `stack` does not support indexing or iteration.
    
6. For problems involving the nearest greater/smaller element, histogram areas, or stock spans, consider a monotonic stack.
    

One-line revision: `stack<T>` follows LIFO, provides access only to the top element, and is essential for DFS, bracket matching, and monotonic-stack algorithms.