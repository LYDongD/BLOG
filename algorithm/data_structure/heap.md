## heap

> 概念 - 实现 - 应用 - 复杂度分析

```

| heap
	| Definition
		| complete binary tree 
			| 除了最后一层，其他层都是满节点
			| 最后一层的节点都靠左
		| 父节点都比子节点大(小)
			| 根节点是最大(最小)节点
				| max-top heap
				| min-top heap
		| feature
			| half leafs and half none-leafs
			| build -> heapify up to down from half
	| Implementation
		| 实现
			| Array
		| insert -> build
			| heapify -> down to up
			| heapify -> up to down
		| remove top
			| heapify -> up to down
		| heapify
			| parent & child -> compare & switch 
			| top & leaf node -> switch

	| Application
		| heap sort
			| Array -> build heap
			| move max/min top to leaf [2]
			| up-down-heapify
			| repeat from [2] 

		| data topK & percent division
		| priority queue
			| java 
				| PriorityQueue & DelayedWorkQueue

	| Complexity
		| heapify -> rely on tree high
			| time: O(logn)
		| buid heap  -> s = sum(num * h）
			| num : 每层节点个数 
			| h : 层高度，从第1层开始(第0层为叶子节点)
			| -> time: O(n)，注意，不是nlogn
			
		| heap sort -> build heap + sort
			| time：O(n) + O(nlogn) = O(logn)
			| space: O(1）in place sort
			| unstable sort
```

> 插入，堆化，移除堆顶元素

```
package heap

type Heap struct {
    a     []int
    n     int
    count int
}

//init heap
func NewHeap(capacity int) *Heap {
    heap := &Heap{}
    heap.n = capacity
    heap.a = make([]int, capacity+1)
    heap.count = 0
    return heap
}

//top-max heap -> heapify from down to up
func (heap *Heap) insert(data int) {
    //defensive
    if heap.count == heap.n {
        return
    }

    heap.count++
    heap.a[heap.count] = data

    //compare with parent node
    i := heap.count
    parent := i / 2
    for parent > 0 && heap.a[parent] < heap.a[i] {
        swap(heap.a, parent, i)
        i = parent
        parent = i / 2
    }
}

//heapfify from up to down
func (heap *iHeap) removeMax() {

    //defensive
    if heap.count == 0 {
        return
    }

    //swap max and last
    swap(heap.a, 1, heap.count)
    heap.count--

    //heapify from up to down
    heapifyUpToDown(heap.a, heap.count)
}

//heapify
func heapifyUpToDown(a []int, count int) {

    for i := 1; i <= count/2; {

        maxIndex := i
        if a[i] < a[i*2] {
            maxIndex = i * 2
        }

        if i*2+1 <= count && a[maxIndex] < a[i*2+1] {
            maxIndex = i*2 + 1
        }

        if maxIndex == i {
            break
        }

        swap(a, i, maxIndex)
        i = maxIndex
    }

}

//swap two elements
func swap(a []int, i int, j int) {
    tmp := a[i]
    a[i] = a[j]
    a[j] = tmp
}

```

> 堆排序

```
package heap
  
//build a heap
func buidHeap(a []int, n int) {

    //heapify from the last parent node
    for i := n / 2; i >= 1; i-- {
        heapifyUpToDown(a, i, n)
    }

}

//sort by ascend, a index begin from 1, has n elements
func sort(a []int, n int) {
    buidHeap(a, n)

    k := n
    for k >= 1 {
        swap(a, 1, k)
        heapifyUpToDown(a, 1, k-1)
        k--
    }
}


//heapify from up to down , node index = top
func heapifyUpToDown(a []int, top int, count int) {

    for i := top; i <= count/2; {

        maxIndex := i
        if a[i] < a[i*2] {
            maxIndex = i * 2
        }

        if i*2+1 <= count && a[maxIndex] < a[i*2+1] {
            maxIndex = i*2 + 1
        }

        if maxIndex == i {
            break
        }

        swap(a, i, maxIndex)
        i = maxIndex
    }

}

//swap two elements
func swap(a []int, i int, j int) {
    tmp := a[i]
    a[i] = a[j]
    a[j] = tmp
}

```

> 关键点总结

* heapify 可以从任意节点开始，即以任意节点作为当前top
* 每次heapify，例如up->down, 都会将下面最大(小）元素交换至顶端(起始)



