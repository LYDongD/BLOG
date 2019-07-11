## 集合实现

### 用数组实现队列

```
type Element interface{}

type Queue interface {
    IsEmpty() bool
    Offer(ele Element) 
    Poll() Element
}

type ListQueue struct {
    elements []Element
}

func NewListQueue() *ListQueue{
   return &ListQueue{}
}

func (queue *ListQueue) IsEmpty() bool {
    return len(queue.elements) == 0
}

func (queue *ListQueue) Offer(e Element) {
    queue.elements = append(queue.elements, e) 
}

func (queue *ListQueue) Poll() Element {
    if queue.IsEmpty() {
        panic("queue is empty")
    }

    ele := queue.elements[0]
    queue.elements = queue.elements[1:]
    return ele
}



```
