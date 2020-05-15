---
title: 数据结构中的链表(Javascript实现)
categories: 数据结构
---


#### 什么是链表
 链表存储有序的元素集合，但不同于数组，链表中的元素在内存中并不是连续放置的。每个元素都由一个存储元素本身的节点和一个指向下一个元素的引用组成。
 
 
 相对于传统的数组，链表有一个好处在于，添加和移除元素的时候不需要移动其他元素。然而，链表需要使用指针，因此使用链表时需要额外注意。
 
 在数组中，
 我们可以直接访问任何位置的任何元素，而想要访问链表中间的一个元素，则需要从起点（表头）开始迭代链表直到找到所需的元素。
 
 #### 单向链表的实现
 ```
 // 链表节点类
class Node {
    constructor(element) {
        this.element = element;
        this.next = undefined;
    }
}


// 链表类
class LinkedList {
    constructor() {
        this.count = 0; // 链表元素数量
        this.head = undefined; // 链表表头元素
    }
    /**
     * 向链表尾部添加元素
     */
    push(element) {
        const node = new Node(element);
        if(!this.head) {
            this.head = node;
        } else {
            let current = this.head;
            while(current.next) { // 迭代访问到链表结尾
                current = current.next;
            }
            current.next = node;
        }
        this.count ++;
    }
    /**
     * 移除指定位置的元素
     */
    removeAt(index) {
        // 检查越界
        if(index >= 0 && index < this.count) {
            let current = this.head;
            if(index === 0) { // 移除表头元素
                this.head = current.next;
            } else {
                let previous;
                for(let i = 0;i < index;i ++) {
                    previous = current;
                    current = current.next;
                }
                // 跳过指定位置的节点，从而达到移除元素的目的
                previous.next = current.next;
            }
            this.count --;
            return current.element;
        }
        return undefined;
    }
    /**
     * 在任意位置添加元素
     */
    insert(element, index) {
        // 检查越界
        if(index >= 0 && index < this.count) {
            const node = new Node(element);
            if(index === 0) { // 在第一个位置添加
                const current = this.head;
                node.next = current;
                this.head = node;
            } else {
                let previous, current = this.head;
                for(let i = 0;i < index;i ++) {
                    previous = current;
                    current = current.next;
                }
                node.next = current;
                previous.next = node;
            }
            this.count ++;
            return true;
        }
        return false;
    }
    /**
     * 返回一个元素的位置
     */
    indexOf(element) {
        let current = this.head;
        for (let i = 0;i < this.count && current != null;i ++) {
            if (element === current.element) {
                return i;
            }
            current = current.next;
        }
        return -1;
    }
    /**
     * 从链表中移除元素
     */
    remove(element) {
        const index = this.indexOf(element);
        return this.removeAt(index);
    }
    /**
     * 获取链表长度
     */
    size() {
        return this.count;
    }
    /**
     * 判断链表是否为空
     */
    isEmpty() {
        return this.count === 0;
    }
    toString() {
        if(!this.head) return '';
        let objString = `${this.head.element}`;
        let current = this.head.next;
        for(let i = 1;i < this.size() && current != null;i ++) {
            objString += `${current.element}`;
            current = current.next;
        }
        return objString;
    }
}

const linkedList = new LinkedList();
linkedList.push(5);
linkedList.push(4);
console.log(linkedList.size()) // 2
console.log(linkedList.toString()) // 54
console.log(linkedList.isEmpty()) // false


 ```

 #### 双向链表的实现

 双向链表和普通链表的区别在于，在链表中，一个节点只有链向下一个节点的链接；而在双向链表中，

 一个链向下一个元素，另一个链向前一个元素。如下图所示：

 ![image](/public/images/双向链表.png)

 ```
 
// 双向链表节点
class DoublyNode {
    constructor(element,next,prev) {
        this.element = element;
        this.next = next;
        this.prev = prev
    }
}

// 双向链表
class DoublyLinkedList {
    constructor() {
        this.head = undefined; // 双向链表表头元素
        this.count = 0; // 双向链表元素个数
        this.tail = undefined; // 双向链表表尾元素
    }
    /**
     * 在任意一个位置插入新元素
     */
    insert(element, index) {
        if(index >= 0 && index <= this.count) {
            const node = new DoublyNode(element);
            let current = this.head;
            if(index === 0) { // 双向链表头部插入节点
                if(this.head == null) {
                    this.head = node;
                    this.tail = node
                } else {
                    node.next = this.head;
                    current.prev = node;
                    this.head = node;
                }
            } else if(index === this.count) { // 双向链表尾部插入节点
                current = this.tail;
                current.next = node;
                node.prev = current;
                this.tail = node;
            } else { // 双向链表中间插入节点
                current = this.head;
                let prev;
                // 找到插入位置的前一个元素
                for(let i = 0;i <= index && current != null;i ++) {
                    prev = current;
                    current = current.next;
                }
                node.next = current;
                current.prev = node;
                prev.next = node;
                node.prev = prev;
            }
            this.count ++;
            return true;
        }
        return false;
    }
    /**
     * 从任意位置移除节点
     */
    removeAt(index) {
        if(index >= 0 && index < this.count) {
            let current = this.head;
            if(index === 0) {
                this.head = current.next;
                if(this.count === 1) { // 如果只有一项，更新tail
                    this.tail = undefined;
                } else {
                    this.head.prev = undefined;
                }
            } else if(index === this.count - 1) { // 移除最后一项
                current = this.tail;
                this.tail = current.prev;
                this.tail.next = undefined;
            } else { // 移除链表中间元素
                let prev
                // 找到要移除元素的前一个元素prev
                for(let i = 0;i <= index && current != null;i ++) {
                    prev = current;
                    current = current.next;
                }
                prev.next = current.next;
                current.next.prev = prev;
            }
            this.count --;
            return current.element;
        }
        return undefined;
    }
}
 ```