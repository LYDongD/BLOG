## Graph 图

> 概念 - 实现 - 应用 - 复杂度分析

```
| Definition

	| 无向图 - 有向图 - 加权有向图
	| vertex + edge
	| data structure
		| adjacent matrix
			| (1,2)  ->  from 1 to0503 2 relation 
		| adjacent table
			| linked list array


| Implemention -> 无向图

	| insert -> add Edge
		| edge -> 2 vertex
		| find index in array for vertex
		| add another vertex to linked list 
	| build map -> insert
		| vertex count -> init array & linked list 
		| insert 
	| search path -> path from begin to the goal vertex
		| BFS
			| use queue 
				| poll & push it’s relative vertex from it’s linked list
				| until goal found or queue is empty
		| DFS
			| recursively & backtracing
				| recurse begin vertex 
				| until found
		| key 
			| use visited[] to avoid repeated path
			| use prev[] to record path and show the path by recursively printing the  prev[]
			| use isFound as an exit flag 

| Application
	| friends relation
	| find different level follower & follows 


| Complexity
	| consider vertex as V, edge as E
	| BFS 
		| space: O(V)
		| time: O(V+E) = O(E)
	| DFS
		| time: O(E)
		| space: O(V)

```

> 具体实现

```
package graph

import (
    "container/list"
    "fmt"
)

//adjacency table, 无向图
type Graph struct {
    adj []*list.List
    v   int
}

//init graphh according to capacity
func newGraph(v int) *Graph {
    graphh := &Graph{}
    graphh.v = v
    graphh.adj = make([]*list.List, v)
    for i := range graphh.adj {
        graphh.adj[i] = list.New()
    }
    return graphh
}

//insert as add edge，一条边存2次
func (self *Graph) addEdge(s int, t int) {
    self.adj[s].PushBack(t)
    self.adj[t].PushBack(s)
}



//search path by BFS
func (self *Graph) BFS(s int, t int) {

    //todo
    if s == t {
        return
    }

    //init prev
    prev := make([]int, self.v)
    for index := range prev {
        prev[index] = -1
    }

    //search by queue
    var queue []int
    visited := make([]bool, self.v)
    queue = append(queue, s)
    visited[s] = true
    isFound := false
    for len(queue) > 0 && !isFound {
        top := queue[0]
        queue = queue[1:]
        linkedlist := self.adj[top]
        for e := linkedlist.Front(); e != nil; e = e.Next() {
            k := e.Value.(int)
            if !visited[k] {
                prev[k] = top
                if k == t {
                    isFound = true
                    break
                }
                queue = append(queue, k)
                visited[k] = true
            }
        }
    }

    if isFound {
        printPrev(prev, s, t)
    } else {
        fmt.Printf("no path found from %d to %d\n", s, t)
    }

}

//search by DFS
func (self *Graph) DFS(s int, t int) {

    prev := make([]int, self.v)
    for i := range prev {
        prev[i] = -1
    }

    visited := make([]bool, self.v)
    visited[s] = true

    isFound := false
    self.recurse(s, t, prev, visited, isFound)

    printPrev(prev, s, t)
}

//recursivly find path
func (self *Graph) recurse(s int, t int, prev []int, visited []bool, isFound bool) {

    if isFound {
        return
    }

    visited[s] = true

    if s == t {
        isFound = true
        return
    }

    linkedlist := self.adj[s]
    for e := linkedlist.Front(); e != nil; e = e.Next() {
        k := e.Value.(int)
        if !visited[k] {
            prev[k] = s
            self.recurse(k, t, prev, visited, false)
        }
    }

}

//print path recursively
func printPrev(prev []int, s int, t int) {

    if t == s || prev[t] == -1 {
        fmt.Printf("%d ", t)
    } else {
        printPrev(prev, s, prev[t])
        fmt.Printf("%d ", t)
    }

}

```


