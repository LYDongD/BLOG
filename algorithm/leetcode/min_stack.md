## Min Stack

### 问题

```

Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

push(x) -- Push element x onto stack.
pop() -- Removes the element on top of the stack.
top() -- Get the top element.
getMin() -- Retrieve the minimum element in the stack.

```

### 思路1

* 使用2个栈，一个保存元素，一个保存最小值记录
* 入栈时，如果元素最小，保存改最小值记录
* 出栈时，如果元素最小，从记录中移除
* 最小值记录栈的栈顶始终保存当前最小元素


```

Stack<Integer> mainStack;
    Stack<Integer> minSelectStack;

    public MinStack() {
        mainStack = new Stack<>();
        minSelectStack = new Stack<>();
    }

    public void push(int x) {

       mainStack.push(x);
       if (minSelectStack.isEmpty() || x <= minSelectStack.peek()){
           minSelectStack.push(x);
       }
    }

    public void pop() {
        if (minSelectStack.peek().equals(mainStack.pop())){
            minSelectStack.pop();
        }
    }

    public int top() {
        return mainStack.peek();
    }

    public int getMin() {
        return minSelectStack.peek();
    }
}

```

### 思路2

* 仅使用一个栈
* 入栈时，确保最小元素的下层是下一个最小元素
* 出栈时，确保最小元素出栈后的顶层元素是下一个最小元素

```

	Stack<Integer> stack;
    int min;

    public MinStack() {
        stack = new Stack<>();
        min = Integer.MAX_VALUE;
    }

    public void push(int x) {

        if (x <= min) {
            stack.push(min);
            min = x;
        }

        stack.push(x);
    }

    public void pop() {
        if (min == stack.pop()) {
            min = stack.pop();
        }
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return min;
    }


```