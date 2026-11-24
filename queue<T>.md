
# C++ Competitive Programming: `queue<T>`

## 1. Short notes (quick revision)

### Definition

A `queue<T>` is an STL container adapter that follows FIFO (First In, First Out). The first element inserted is the first element removed.

```
#include <bits/stdc++.h>
using namespace std;

queue<int> q;

q.push(10);
q.push(20);
q.push(30);

cout << q.front();  // 10
cout << q.back();   // 30

q.pop();            // Removes 10
cout << q.front();  // 20
```

### Important operations

|Operation|Purpose|Complexity|
|---|---|---|
|`q.push(x)`|Insert at the back|\(O(1)\)|
|`q.emplace(x)`|Construct an element at the back|\(O(1)\)|
|`q.pop()`|Remove the front element|\(O(1)\)|
|`q.front()`|Access the first element|\(O(1)\)|
|`q.back()`|Access the last element|\(O(1)\)|
|`q.size()`|Number of elements|\(O(1)\)|
|`q.empty()`|Check whether empty|\(O(1)\)|
|`q.swap(other)`|Swap two queues|Usually \(O(1)\)|

Remember: `pop()` removes an element but does not return it.

```
int x = q.front();
q.pop();
```

### Common competitive programming uses

- BFS (Breadth-First Search)
    
- Shortest path in unweighted graphs
    
- Level-order tree traversal
    
- Grid traversal
    
- Simulation and scheduling
    
- Processing elements in arrival order
    

## 2. Long notes (complete explanation)

### A. Declaration and initialization

```
queue<int> q;
queue<string> names;
queue<pair<int, int>> points;
queue<long long> distances;
```

A queue uses `deque` as its default underlying container.

You can also specify the underlying container:

```
queue<int, list<int>> q;
```

The underlying container must support the operations required by the queue, including insertion at the back and removal from the front.

### B. Insertion and removal

```
queue<int> q;

q.push(10);
q.push(20);
q.push(30);

// Queue: front -> 10, 20, 30 <- back

q.pop();

// Queue: front -> 20, 30 <- back
```

To process all elements:

```
while (!q.empty()) {
    cout << q.front() << " ";
    q.pop();
}
```

Output:

```
20 30
```

The exact output depends on whether any elements were removed before this loop.

### C. Accessing front and back

```
queue<int> q;

q.push(5);
q.push(15);
q.push(25);

cout << q.front();  // 5
cout << q.back();   // 25
```

- `front()` accesses the oldest element.
    
- `back()` accesses the newest element.
    
- Both require the queue to be non-empty.
    

Calling `front()`, `back()`, or `pop()` on an empty queue is invalid.

### D. Checking size and emptiness

```
queue<int> q;

cout << q.empty();  // 1 (true)

q.push(100);
q.push(200);

cout << q.size();   // 2
cout << q.empty();  // 0 (false)
```

A safe processing pattern is:

```
while (!q.empty()) {
    int x = q.front();
    q.pop();

    // Process x
}
```

### E. Queue with pairs

Queues can store pairs, which is particularly useful in graph and grid problems.

```
queue<pair<int, int>> q;

q.push({0, 0});
q.push({1, 2});

auto [x, y] = q.front();
q.pop();

cout << x << " " << y;  // 0 0
```

The structured binding syntax requires C++17.

### F. BFS using a queue

BFS explores vertices level by level and is commonly used to find shortest paths in unweighted graphs.

```
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    vector<vector<int>> adj(n);
    adj[0] = {1, 2};
    adj[1] = {3};
    adj[2] = {4};

    vector<bool> visited(n, false);
    queue<int> q;

    q.push(0);
    visited[0] = true;

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        cout << u << " ";

        for (int v : adj[u]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
}
```

Output:

```
0 1 2 3 4
```

Why mark a node as visited when pushing it? This prevents the same node from being inserted multiple times before it is processed.

Complexity for an adjacency-list graph:

- Time: \(O(V+E)\)
    
- Space: \(O(V)\)
    

Here, \(V\) is the number of vertices and \(E\) is the number of edges.

### G. Shortest path in an unweighted graph

BFS can calculate the shortest distance from a source vertex to every reachable vertex.

```
vector<int> dist(n, -1);
queue<int> q;

q.push(src);
dist[src] = 0;

while (!q.empty()) {
    int u = q.front();
    q.pop();

    for (int v : adj[u]) {
        if (dist[v] == -1) {
            dist[v] = dist[u] + 1;
            q.push(v);
        }
    }
}
```

- `dist[src] = 0`: the source has distance zero from itself.
    
- `dist[v] == -1`: the vertex has not been visited.
    
- `dist[v] = dist[u] + 1`: each edge adds one to the distance.
    

This finds shortest paths by number of edges, not by total weight in a weighted graph.

### H. Level-order traversal in a binary tree

A queue is the standard data structure for level-order traversal.

```
queue<TreeNode*> q;
q.push(root);

while (!q.empty()) {
    TreeNode* node = q.front();
    q.pop();

    cout << node->val << " ";

    if (node->left)
        q.push(node->left);

    if (node->right)
        q.push(node->right);
}
```

Assume `TreeNode` has `val`, `left`, and `right` members and `root` is a valid pointer. Check `root` before inserting it if it may be null.

### I. Clearing and swapping queues

The standard queue interface has no `clear()` member. A common way to empty a queue is:

```
while (!q.empty())
    q.pop();
```

To swap queues:

```
queue<int> a, b;

a.push(1);
a.push(2);
b.push(10);

a.swap(b);
```

After swapping, `a` contains `10`, while `b` contains `1, 2`.

## 3. Queue vs. other STL containers

|Feature|`queue`|`stack`|`deque`|
|---|---|---|---|
|Principle|FIFO|LIFO|Double-ended|
|Insert|Back|Top|Front or back|
|Remove|Front|Top|Front or back|
|Access front/back|Yes|No|Yes|
|Random indexing|No|No|Yes|

Use `queue` for BFS and FIFO processing, `stack` for DFS-style iterative traversal, and `deque` when you need operations at both ends.

## 4. Competitive programming checklist

- Use `queue<int>` for ordinary BFS.
    
- Use `queue<pair<int,int>>` for grid coordinates or `(node, distance)` states.
    
- Mark visited nodes when inserting them into the queue.
    
- Always check `!q.empty()` before accessing or removing elements.
    
- Remember that `q.pop()` does not return the removed value.
    
- Use `queue` for unweighted shortest paths; use Dijkstra's algorithm with a min-priority queue for graphs with non-negative edge weights.
    

One-line revision: `queue<T>` follows FIFO, supports insertion at the back and removal from the front, and is essential for BFS and level-by-level processing.
