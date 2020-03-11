## 面试题30：包含min函数的栈
**题目：定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。**

**思路：除了正常保存的stack之外，使用一个辅助stack来保存每次pop时元素和辅助stack栈顶元素的较小值（输入元素大于栈顶元素则不操作）。取min时时间复杂度O(1)。**
```
public class MinStack {
	/** initialize your data structure here. */
	Stack<Integer> data, orderedData;
    public MinStack() {
    	this.data = new Stack<>();
    	this.orderedData = new Stack<>();
    }
    
    public void push(int x) {
    	this.data.add(x);
    	if(orderedData.empty() || orderedData.peek()>=x) {
    		orderedData.add(x);
    	}
    }
    
    public void pop() {
    	if(data.pop().equals(orderedData.peek())) {
    		orderedData.pop();
    	}
    }
    
    public int top() {
    	return data.peek();
    }
    
    public int min() {
    	return orderedData.peek();
    }
}
```
**时间击败24。。。都已经是O(1)了。。。**