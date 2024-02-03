# Critical Connections in a Network

## Description

There are `n` servers numbered from `0` to `n - 1` connected by undirected server-to-server `connections` forming a network where `connections[i] = [ai, bi]` represents a connection between servers `ai` and `bi`. Any server can reach other servers directly or indirectly through the network.

A *critical connection* is a connection that, if removed, will make some servers unable to reach some other server.

Return all critical connections in the network in any order.

## Algorithm

We can consider this network as an undirected graph, where connections corresponds to edges. In the graph, an edge is critical *if and only if* it is not in a cycle. So, we need to determine whether each edge is in a cycle.

It can be achieved by depth-first search (DFS). When visiting $v$ after visiting $u$, if we find $v$ has been visited, we can confirm that $(u,v)$ is in a cycle. But in order to let previous nodes know that we find a cycle, we need to return something.

Let's consider the depth of each node in the DFS tree. When we just encounter a visited node, it must have smaller depth. The cycle is: `d -> d + 1 -> d + 2 -> ... -> d + k -> d`. As a result, all edges between nodes whose depth is larger than `d` is in the cycle. We just need to return `d`. Generally, we can return the minimum depth of nodes which we have ever encounters.

## Code

```cpp
int dfs(vector<unordered_set<int>>& graph, vector<int>& depth, int v, int depth_v, vector<vector<int>>& critical)
{
    if (depth[v] >= 0)
        return depth[v];
    depth[v] = depth_v;
    int min_depth_below = depth_v;
    for (const auto& next: graph[v])
    {
        graph[next].erase(graph[next].find(v));
        int min_depth_next = dfs(graph, depth, next, depth_v + 1,
            critical);
        min_depth_below = min(min_depth_below, min_depth_next);
        if (min_depth_next > depth_v)
            critical.push_back({v, next});
    }
    return min_depth_below;
}

vector<vector<int>> criticalConnections(int n, 
    vector<vector<int>>& connections)
{
    vector<unordered_set<int>> graph(n);
    for (const auto& connection: connections)
    {
        graph[connection[0]].insert(connection[1]);
        graph[connection[1]].insert(connection[0]);
    }
    vector<int> depth(n, -1);
    vector<vector<int>> critical;
    dfs(graph, depth, 0, 0, critical); // DFS from node 0, whose depth is 0

    return critical;
}
```