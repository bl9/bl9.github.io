+++
date = '2025-12-07T14:55:06-05:00'
draft = false
categories = ['algorithms', 'graphs']
title = 'Ucs Shortest Path Algorithm'
+++

UCS is a shortest path algorith that's similar to Dijkstra algorithm. The only difference is that UCS will terminate as soon as it finds the goal node, while Dijkstra will continue to explore all possible paths until it has found the shortest path to all nodes in the graph.

So based on that, to implement UCS, we can use a priority queue to keep track of the nodes to be explored, sorted by their cumulative cost from the start node. We will also maintain a set of visited nodes to avoid processing the same node multiple times.

To summarize the steps of UCS algorithm:
1. Initialize a priority queue with the start node and a cumulative cost of 0.
2. While the priority queue is not empty:
   a. Dequeue the node with the lowest cumulative cost.
    b. If the node is the goal node, return the cumulative cost as the shortest path cost.
    c. If the node has not been visited:
         i. Mark the node as visited.
        ii. For each neighbor of the node, calculate the cumulative cost to reach that neighbor and enqueue it in the priority queue.

Here's an example for simple UCS implementation in python:

```python
import heapq
from collections import defaultdict
class Graph:
    def __init__(self):
        self.edges = defaultdict(list)
        self.weights = {}

    def add_edge(self, from_node, to_node, weight):
        self.edges[from_node].append(to_node)
        self.weights[(from_node, to_node)] = weight
def ucs(graph, start, goal):
    queue = [(0, start)]
    visited = set()
    while queue:
        (cost, node) = heapq.heappop(queue)
        if node in visited:
            continue
        visited.add(node)
        if node == goal:
            return cost
        for neighbor in graph.edges[node]:
            if neighbor not in visited:
                total_cost = cost + graph.weights[(node, neighbor)]
                heapq.heappush(queue, (total_cost, neighbor))
    return float("inf")
# Example usage
g = Graph()
g.add_edge('A', 'B', 1)
g.add_edge('A', 'C', 4)
g.add_edge('B', 'C', 2)
g.add_edge('B', 'D', 5)
g.add_edge('C', 'D', 1)
start = 'A'
goal = 'D'
cost = ucs(g, start, goal)
print(f"The cost of the shortest path from {start} to {goal} is {cost}.")
```

