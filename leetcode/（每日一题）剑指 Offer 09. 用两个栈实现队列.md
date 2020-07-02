## （每日一题）剑指 Offer 09. 用两个栈实现队列

**难度**：

简单

**题目**

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

**示例1**：

>输入：
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]

**示例2**：

>输入：
["CQueue","deleteHead","appendTail","appendTail","deleteHead","deleteHead"]
[[],[],[5],[2],[],[]]
输出：[null,-1,null,null,5,2]

**提示**：

* 1 <= values <= 10000
* 最多会对 appendTail、deleteHead 进行 10000 次调用

**思路**：

使用两个栈 A 和 B。appendTail 时将元素直接添加到 B 中，deleteHead 时，判断 A 是否为空，空的话就把 B 中的元素全部压入 A 中再返回，如果还是为空则返回 -1。

```java
class CQueue {

    Stack<Integer> stackA;
	Stack<Integer> stackB;

    public CQueue() {
    	stackA = new Stack<>();
    	stackB = new Stack<>();
    }
    
    public void appendTail(int value) {
    	stackB.push(value);
    }
    
    public int deleteHead() {
    	if(stackA.isEmpty()) {
    		while(!stackB.isEmpty()) {
    			stackA.push(stackB.pop());
    		}
    	}
    	if(stackA.isEmpty()) {
    		return -1;
    	}else {
			return stackA.pop();
		}
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```