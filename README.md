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

    // =================================================================
    // FUNGSI BANTUAN 1: Trace Next Hop
    // =================================================================
    long get_next_hop(long start, long target, const vector<long>& p) {
        if (target == start) return start;
        if (p[target] == -1) return -1;
        long curr = target;
        while (p[curr] != start && p[curr] != -1) curr = p[curr];
        return curr;
    }

    // =================================================================
    // FUNGSI BANTUAN 2: Hitung Total Weight dari Jalur Parent
    // =================================================================
    long calculate_path_weight(long start, long target, const vector<long>& p) {
        if (start == target) return 0;
        if (p[target] == -1) return INF;

        long total_cost = 0;
        long curr = target;

        while (curr != start) {
            long prev = p[curr];
            if (prev == -1) return INF; 

            long weight_found = INF;
            // Loop manual pengganti auto untuk compiler lama
            for(size_t i=0; i<adjList[prev].size(); i++) {
                if (adjList[prev][i].first == curr) {
                    weight_found = adjList[prev][i].second;
                    break;
                }
            }
            
            if (weight_found == INF) return INF;
            
            total_cost += weight_found;
            if (total_cost >= INF) return INF;

            curr = prev;
        }
        return total_cost;
    }

    // =================================================================
    // 1. BFS 
    // =================================================================
    void run_bfs(long start) {
        if(!is_active[start]) return;
        vector<long> hops(vertexCount, INF);
        vector<long> p(vertexCount, -1);
        queue<int> q;

        hops[start] = 0; q.push(start);

        while(!q.empty()){
            int u = q.front(); q.pop();
            for(size_t i=0; i<adjList[u].size(); i++){
                int v = adjList[u][i].first;
                if(is_active[v] && hops[v] == INF){ 
                    hops[v] = hops[u] + 1; 
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

    // =================================================================
    // 2. DIJKSTRA 
    // =================================================================
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

    // =================================================================
    // 3. BELLMAN-FORD / RIP 
    // =================================================================
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
    g.init(6); // Inisialisasi 6 node
    g.add_edge(0, 1, 2); g.add_edge(0, 5, 4);
    g.add_edge(1, 3, 9); g.add_edge(5, 3, 2);
    g.add_edge(3, 4, 1); g.add_edge(3, 2, 5);
    
    for(int i=0; i<50; i++) g.run_rip_step();

    long my_router = 0; // Pantau dari R0

    while(true) {
        cout << "\n=== DASHBOARD ROUTER " << my_router << " ===\n";
        for(int i=0; i<g.vertexCount; i++) cout << "R" << i << (g.is_active[i]?"[ON] ":"[OFF] ");
        
        cout << "\n\nDEST | COST | NEXT HOP\n";
        cout << "-----+------+---------\n";
        for(int i=0; i<g.vertexCount; i++) {
            pair<long, long> r = g.tables[my_router][i];
            
            // Konversi manual to_string (kalau compiler sangat tua & tidak support to_string)
            // Tapi biasanya C++11 support to_string. Jika error 'to_string' not declared, 
            // bilang saja, nanti saya ganti pakai stringstream.
            
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
    

- a
