# final_project_graph_group_1

Group Number: 1 <br>

Group Members:
-	Alif Muflih Jauhary (5025241003)
-	Rayen Yeriel Mangiwa (5025241262)
-	Makna Alam Pratama (5025241077)

## The Program
```
#include <bits/stdc++.h>
using namespace std;

const long INF = 20; 

struct graph {
    long vertexCount;
    // Perhatikan spasi di antara > > supaya aman di compiler tua
    vector<vector<pair<long, long> > > adjList; 
    vector<bool> is_active; 
    
    // TABLES: pair<COST, NEXT_HOP>
    vector<vector<pair<long, long> > > tables;

    void init(long v){
        vertexCount = v;
        adjList.clear(); adjList.resize(v);
        is_active.assign(v, true);
        
        tables.resize(v, vector<pair<long, long> >(v));
        
        // --- PERBAIKAN DI SINI (Ganti Ternary dengan If-Else) ---
        for(int i=0; i<v; i++) {
            for(int j=0; j<v; j++) {
                if(i == j) {
                    // Jarak ke diri sendiri = 0, Next Hop = diri sendiri
                    // Kita casting (long) supaya tipe datanya pas
                    tables[i][j] = make_pair((long)0, (long)i);
                } else {
                    // Jarak ke orang lain = INF, Next Hop = -1
                    tables[i][j] = make_pair(INF, (long)-1);
                }
            }
        }
    }

    void add_edge(long v1, long v2, long w){
        adjList[v1].push_back(make_pair(v2, w));
        adjList[v2].push_back(make_pair(v1, w)); 
    }

    void set_status(long node, bool status) {
        if(node >= 0 && node < vertexCount) is_active[node] = status;
    }

    long get_next_hop(long start, long target, const vector<long>& p) {
        if (target == start) return start;
        if (p[target] == -1) return -1;
        long curr = target;
        while (p[curr] != start && p[curr] != -1) curr = p[curr]; // stop before the condition met
        return curr;
    }

    long calculate_path_weight(long start, long target, const vector<long>& p) {
        if (start == target) return 0;
        if (p[target] == -1) return INF;

        long total_cost = 0;
        long curr = target;

        while (curr != start) {
            long prev = p[curr]; // this is the parent
            if (prev == -1) return INF; 

            long weight_found = INF;
            for(size_t i=0; i<adjList[prev].size(); i++) { // check using the parent
                if (adjList[prev][i].first == curr) {
                    weight_found = adjList[prev][i].second;
                    break;
                }
            }
            
            if (weight_found == INF) return INF;
            
            total_cost += weight_found;
            if (total_cost >= INF) return INF;

            curr = prev; // taking the parent means taking the previous node
        }
        return total_cost;
    }

    // 1. BFS 
    void run_bfs(long start) {
        if(!is_active[start]) return;
        vector<long> p(vertexCount, -1);
        queue<int> q;

        q.push(start);
        while(!q.empty()){
            int u = q.front(); q.pop();
            for(size_t i=0; i<adjList[u].size(); i++){
                int v = adjList[u][i].first;
                if(is_active[v]){
                    p[v] = u; 
                    q.push(v);
                }
            }
        }
        
        for(int i=0; i<vertexCount; i++) {
            tables[start][i].first = calculate_path_weight(start, i, p);
            tables[start][i].second = get_next_hop(start, i, p);
        }
        cout << ">>> [BFS] Table ditimpa (Path: Min Hop | Cost: Real Weight).\n";
    }

    // 2. DIJKSTRA 
    void run_dijkstra(long start) {
        if(!is_active[start]) return;
        vector<long> d(vertexCount, INF), p(vertexCount, -1);
        
        // Priority Queue Syntax aman untuk compiler lama
        priority_queue<pair<long,long>, vector<pair<long,long> >, greater<pair<long,long> > > pq;

        d[start] = 0; pq.push(make_pair(0, start));
        vector<bool> vis(vertexCount, false);

        while(!pq.empty()){
            long u = pq.top().second; pq.pop();
            if(vis[u]) continue; vis[u] = true;

            for(size_t i=0; i<adjList[u].size(); i++){
                long v = adjList[u][i].first;
                long w = adjList[u][i].second;
                if(is_active[v] && d[u] + w < d[v]){
                    d[v] = d[u] + w;
                    p[v] = u;
                    pq.push(make_pair(d[v], v));
                }
            }
        }

        for(int i=0; i<vertexCount; i++) {
            tables[start][i].first = d[i];
            tables[start][i].second = get_next_hop(start, i, p);
        }
        cout << ">>> [DIJKSTRA] Table ditimpa (Path: Min Cost | Cost: Real Weight).\n";
    }

    // 3. BELLMAN-FORD / RIP 
    bool run_rip_step() {
        bool changed = false;
        vector<vector<pair<long, long> > > old = tables; 

        for(long u=0; u<vertexCount; u++) {
            if(!is_active[u]) continue; 

            for(long dest=0; dest<vertexCount; dest++) {
                if(u == dest) continue;

                for(size_t i=0; i<adjList[u].size(); i++) {
                    long v = adjList[u][i].first;
                    long w = adjList[u][i].second;

                    long link_cost = is_active[v] ? w : INF;
                    long total_cost = link_cost + old[v][dest].first; 
                    if(total_cost > INF) total_cost = INF;

                    if (total_cost < tables[u][dest].first) {
                        tables[u][dest] = make_pair(total_cost, v);
                        changed = true;
                    }
                    else if (tables[u][dest].second == v && total_cost != tables[u][dest].first) {
                        tables[u][dest].first = total_cost;
                        changed = true;
                    }
                }
            }
        }
        return changed;
    }
};

int main(){
    graph g;
    graph g;
    long V, E;
    
    cout << "Masukkan jumlah vertex: ";
    cin >> V;

    cout << "Masukkan jumlah edge: ";
    cin >> E;

    g.init(V);

    cout << "Masukkan edge dalam format: u v w\n";
    cout << "(u dan v adalah vertex, w adalah weight)\n";
    for(long i = 0; i < E; i++){
        long u, v, w;
        cin >> u >> v >> w;
        g.add_edge(u, v, w);
    }
    // g.init(6); // Inisialisasi 6 node
    // g.add_edge(0, 1, 2); g.add_edge(0, 5, 4);
    // g.add_edge(1, 3, 9); g.add_edge(5, 3, 2);
    // g.add_edge(3, 4, 1); g.add_edge(3, 2, 5);
    
    for(int i=0; i<50; i++) g.run_rip_step();

    long my_router = 0; // Pantau dari R0

    while(true) {
        cout << "\n=== DASHBOARD ROUTER " << my_router << " ===\n";
        for(int i=0; i<g.vertexCount; i++) cout << "R" << i << (g.is_active[i]?"[ON] ":"[OFF] ");
        
        cout << "\n\nDEST | COST | NEXT HOP\n";
        cout << "-----+------+---------\n";
        for(int i=0; i<g.vertexCount; i++) {
            pair<long, long> r = g.tables[my_router][i];
            
            
            string c = (r.first >= INF) ? "INF" : to_string(r.first);
            string n;
            if(r.second == -1) n = "-";
            else if(r.second == my_router) n = "SELF";
            else n = "R" + to_string(r.second);
            
            cout << "R" << i << "   | " << c << "    | " << n << endl;
        }

        cout << "\n[1] Step RIP (Looping)\n[2] Dijkstra (Reset Cost)\n[3] BFS (Reset Hop)\n[4] Toggle Router\n[5] Ganti Pantauan\n>> ";
        int pil; cin >> pil;

        if(pil==1) { 
            if(!g.run_rip_step()) cout << "Stabil.\n"; 
            else cout << "Update Terjadi!\n"; 
        }
        else if(pil==2) g.run_dijkstra(my_router);
        else if(pil==3) g.run_bfs(my_router);
        else if(pil==4) { int t; cout<<"ID: "; cin>>t; g.set_status(t, !g.is_active[t]); }
        else if(pil==5) { cout<<"ID: "; cin>>my_router; }
        else break;
    }
    return 0;
}
```

# Specific Explanation

### main variables
```
vector<vector<pair<long, long>>> adjList;
vector<vector<pair<long, long>>> tables;
```
- pair<long, long> represents one edge:
  ```
  pair<long, long> = {neighborIndex, weight};
  ```
- vector<pair<long, long>> means a list of neighbors for one vertex:
  ```
  vector<pair<long,long>> neighbors = { {1, 5}, {4, 12} };
  ```
- vector<vector<pair<long, long>>> is the adjacency list for the entire graph:
  ```
  vector<vector<pair<long, long>>> adjList = [
      [ (1,5), (3,10) ],     // neighbors of router 0
      [ (0,5) ],             // neighbors of router 1
      [ (5,2), (4,8) ],      // neighbors of router 2
      ...
  ]
  ```
  So: <br>
      - adjList[0] → all connections from router 0 <br>
      - adjList[0][0] → the first neighbor of router 0, (1, 5) <br>
      - adjList[0][0].first → 1 (neighbor) <br>
      - adjList[0][0].second → 5 (weight) <br>
  However, we manage it differently for the routing table:
  ```
  tables[u][v] = { cost_to_v , nextHop_from_u }
  ```
  Example: routing table of router id = 0
  ```
  tables[0] = [
        (0,0),  // to 0 self
        (2,1),  // to 1 cost 2
        (7,3),  // to 2 cost 7
        (4,5),  // to 3 cost 4
        (6,5),  // to 4 cost 6
        (4,5)   // to 5 cost 4
  ]
  ```
  Why they look structurally different? Because table is always full while    adjList able to have different spaces
### init
Purpose:
- to make V initial rows for the edges to be filled:
  ```
  adjList.resize(v);
  ```
- to initiate V × V matrix for routing table: <br>
  The initiation as {0, 0}
  ```
  tables.resize(v, vector<pair<long, long>>(v));
  ```
  Setting the hop to itself by {0, i}
  ```
  if (i == j) {
    tables[i][j] = make_pair((long)0, (long)i);
  }
  ```
  So:
    ```
    tables[0][0] = {0, 0}
    tables[1][1] = {0, 1}
    tables[2][2] = {0, 2}
    ```
  Else (i != j) → unreachable at the start
  ```
  tables[i][j] = make_pair(INF, (long)-1);
  ```
  So:
    ```
    tables[0][1] = {INF, -1}
    tables[0][2] = {INF, -1}
    tables[1][0] = {INF, -1}
    tables[2][0] = {INF, -1}
    ...
    ```
    
### get_next_hop
Variable p → a parent array from a shortest-path algorithm
```
p[x] = y
```
means: To reach x, you came from y. <br>
Example:
    ```
    p = [0, 0, 1, 2, 3]
    ```
    which: <br>
    - p[0] = 0 <br>
    - p[1] = 0 <br>
    - p[2] = 1 <br>
    - ... <br>
    means the path is: 0 → 1 → 2 → 3 → 4 <br>
    which is `p[child] = parent`
What you'll return is the next hop

```cpp
// -1 (no parent)
long curr = target;
while (p[curr] != start && p[curr] != -1)
    curr = p[curr];
return curr;
```

This loop does:

```
curr = target
curr = p[target]
curr = p[p[target]]
curr = p[p[p[target]]]
...
```

Because `const vector<long>& p` acts as a reference to the local variable that the reference gets passed on.

### calculate_path_weight
Follow the parents (p) from target → start; just as get_next_hop function. <br>
Conditions:
- If start == target → cost is 0
- If p[target] = -1 → target unreachable
- If p[curr] = -1 in while (curr != start) → target unreachable
- If parent adjList met the connection with the child → take the value to check the validity even more before returning the value

### run_bfs
Prepare arrays for BFS
```
vector<long> hops(vertexCount, INF); // for the routing table
vector<long> p(vertexCount, -1);
queue<int> q;
```
The BFS loop
1. Take the front node u.
   ```
   while(!q.empty()) {
       int u = q.front(); q.pop();
   ```
2. Explore all neighbors of u (v = a neighbor of u):
   ```
   for(size_t i=0; i<adjList[u].size(); i++){
       int v = adjList[u][i].first;
   ```
3. Check if this neighbor v should be visited <br>
   - is_active[v] → router is alive <br>
   - p[v] = u → remember who discovered v
   - q.push(v) → add for further exploration
4. After BFS is complete → fill routing table row

### run_dijkstra
Prepare arrays for Dijkstra
```
vector<long> d(vertexCount, INF), p(vertexCount, -1);
```
Create an ascending priority queue
```
priority_queue<pair<long,long>, vector<pair<long,long>>, greater<pair<long,long>>> pq;
```
which, each element is: `(current_distance, node_id)` <br>
The Dijkstra loop:
1. u = pq.top().second → pick node with minimum cost so far
2. Explore all neighbors of u
   ```
   for(size_t i=0; i<adjList[u].size(); i++){
    long v = adjList[u][i].first;
    long w = adjList[u][i].second;
   ```
4. Check is_active[v] && d[u] + w < d[v] → only process neighbors that are  && found a cheaper path to reach v
   ```
   d[v] = d[u] + w;
   p[v] = u;
   pq.push(make_pair(d[v], v));
   ```
   - d[v] = d[u] + w → update new best distance <br>
   - p[v] = u → store u as the parent of v <br>
   - pq.push(make_pair(d[v], v)) → pushing the improved distance into queue <br>

After Dijkstra is complete → fill routing table row
1. tables[start][i].first = d[i] → store minimum cost from start to i
2. tables[start][i].second = get_next_hop(start, i, p) → use the parent array to reconstruct

### run_rip_step
What this do:  <br>
"Every router u updates its routing table by learning from its neighbors' tables." <br>
which literally:
```
For each router u:
    For each destination dest:
        For each neighbor v of u:
            Compute: u → v → dest
            If cheaper: update u's table
```
First
```
changed = false
```
→ we will return true if ANY routing table entry gets improved (means: “a change occurred in this iteration”) <br>
<br>
Make a COPY of the tables
```
vector<vector<pair<long, long> > > old = tables;
```
→ RIP must use old tables from neighbors. This matches RIP rule: "use the neighbors’ last advertised table, not their in-progress updates." <br>
Loops
1. Outer loop 1 — each router u
   ```
   for(long u = 0; u < vertexCount; u++) {
       if(!is_active[u]) continue;
   ```
   → Skip router if inactive
2. Outer loop 2 — each destination dest
   ```
   for(long dest = 0; dest < vertexCount; dest++) {
    if(u == dest) continue;
   ```
   → Skip dest == u because distance to itself is always 0
3. Inner loop — examine neighbors v of u
   ```
   for(size_t i=0; i < adjList[u].size(); i++) {
       long v = adjList[u][i].first;
       long w = adjList[u][i].second;
   ```
   → w = link weight of edge (u → v)
4. Compute link cost (in case neighbor is down)
   ```
   long link_cost = is_active[v] ? w : INF;
   ```
   - If neighbor v is active → cost = w <br>
   - If v is dead → treat link as unreachable (INF) <br>
5. Compute total cost using the neighbor’s table

```
long total_cost = link_cost + old[v][dest].first;
if (total_cost > INF) total_cost = INF;
```

Remember that:

```
adjList[u]                  → list of neighbors of u
adjList[u][i].first        → neighbor node v
adjList[u][i].second       → cost(u → v)
```

and

```
tables[u][dest].first      → cost(u → dest)
tables[u][dest].second     → next hop from u toward dest
```

Think of it like this:

```
u ───(link cost)──▶ v ───(v's known cost to dest)──▶ dest
```

- The link between u and v:

    ```
    1 ── cost = 3 ──▶ 2
    ```

- v knows about reaching destination 7:

    ```
    2 → 7 costs 5
    ```

  meaning:

```
old[2][7] = (5, nextHopTo7)
  ↑
.first
```

---

6. Compare with current table — improvement check

```
if (total_cost < tables[u][dest].first) {
    tables[u][dest] = make_pair(total_cost, v);
    changed = true;
}
```

What we're going to implement:

Imagine 4 routers:

```
0 --(1)-- 1 --(1)-- 2 --(1)-- 3
```

- **Iteration 1**: Router 0 asks neighbor 1:

    ```
    0 → 1 = 1
    1 → 2 = 1
    So maybe 0 → 2 = 1 + 1 = 2
    ```

- **Iteration 2**: Router 0 asks neighbor 1 again:

    ```
    0 → 1 = 1
    1 → 3 = 2  (found last time)
    So maybe 0 → 3 = 1 + 2 = 3
    ```

So little by little, routers learn distances across the network.

7. Second condition: route via v is outdated

```
else if (tables[u][dest].second == v && total_cost != tables[u][dest].first) {
    tables[u][dest].first = total_cost;
    changed = true;
}
```

`tables[u][dest].second == v && total_cost != tables[u][dest].first`  
→ The cost to reach `dest` **through v** changed.

Example:

Before:

```
1 → 3 → 7 cost = 12
```

But now neighbor 3 says its cost to 7 changed:

```
new total cost = 15   (for example)
```

So:

```
total_cost (15) != tables[1][7].first (12)
```

Which means:

- cost is different than before  
- the route is outdated  

---

# HOW COUNT-TO-INFINITY HAPPENS

```
A —1— B —1— C
```

Router C suddenly goes down.

---

🟦 **Step 1 — B notices C is down**

So B sets:

```
B → C = INF
```

But A does NOT know this yet.

---

🟦 **Step 2 — A sends its old table to B**

A still believes:

```
A → C cost = 2 via B
```

And RIP rule says:

```
Distance(B→C) = weight(B→A) + Distance(A→C)
```

So B computes:

```
B → C = 1 + 2 = 3
```

B says:

```
“Oh, A thinks C is reachable!
I guess I can reach C through A.”
```

Even though both routes go through each other → **LOOP**.

---

🟥 **Step 3 — B updates its table**

```
B → C = 3 via A
```

---

🟦 **Step 4 — A receives B’s update**

A sees:

```
B → C = 3
```

So A computes:

```
A → C = 1 + 3 = 4
```

Now:

```
A → C = 4 via B
```

---

# 🟥 The Loop Continues

Next update:

B sees A → C = 4:

```
B → C = 1 + 4 = 5
```

Then A sees B → C = 5:

```
A → C = 1 + 5 = 6
```

This keeps going:

```
3
4
5
6
7
8
...
16
∞
```

This is the **Count-to-Infinity problem**.  
RIP increments route costs one hop at a time until reaching INF.

---

# 🟥 Which part of YOUR CODE causes it?

This part:

```
long total_cost = link_cost + old[v][dest].first;
if (total_cost > INF) total_cost = INF;
```

---

# 🟥 Why does RIP do this?

Because RIP:

- trusts neighbors  
- does not detect loops  
- updates cost slowly (1 hop at a time)  

