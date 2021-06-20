这里我们只看单链表，单链表的每个节点都保存了一个数据项和一个指向下一个节点的next指针，所以我们先要定义一个节点(Node)类，然后再来看看链表。

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None
```

下面看看如何实现列表增加，删除，插入，更新，查找节点操作。

```python
# 在index出插入数据项为data的节点
# 考虑三种情况：链表头部，尾部，中间插入
def insert(self, data, index):

    if index < 0 or index > self.size:
        raise Exception('超了')

    node = Node(data)
    # 插入第一个节点时设置链表的头节点和尾结点
    if self.size == 0:
        self.head = node
        self.tail = node
    # 在头部插入
    # 1. 将该节点的next指向原头节点
    # 2. 头结点的引用指向该节点
    elif index == 0:
        node.next = self.head
        self.head = node
    # 在尾部插入
    # 1. 将尾结点的next指向该节点
    # 2. 将尾结点的引用指向该节点
    elif index == self.size:
        self.tail.next = node
        self.tail = node
    # 在中间插入
    # 1. 找到index的前一个节点preNode
    # 2. 将该节点的next指向preNode的下一个节点，即preNode.next
    # 3. 将preNode的next指向该节点
    # 注意：第2跟第3不能颠倒
    else:
        preNode = self.get(index-1)
        node.next = preNode.next
        preNode.next = node
    self.size = self.size + 1
```

链表中有一个head和tail属性总是指向链表的头和尾节点，还有一个size表示当前节点的个数。如果当前size为0，可插入节点索引index的范围只能是0，如果size为5，index的范围为[0,5]，虽然节点的索引范围是[0,4]。

下面是完整代码

```python
class ListNode:
    def __init__(self):
        self.size = 0
        self.head = None
        self.tail = None

    # index必须为合法的索引值
    # 查找索引index出的节点
    def get(self, index):
        if index < 0 or index > self.size - 1:
            raise Exception('超出索引范围')
        node = self.head
        if node is None:
            return None
        for i in range(index):
            node = node.next
        return node

    # 在index出插入数据项为data的节点
    # 考虑三种情况：链表头部，尾部，中间插入
    def insert(self, data, index):

        if index < 0 or index > self.size:
            raise Exception('超了')

        node = Node(data)
        # 插入第一个节点时设置链表的头节点和尾结点
        if self.size == 0:
            self.head = node
            self.tail = node
        # 在头部插入
        # 1. 将该节点的next指向原头节点
        # 2. 头结点的引用指向该节点
        elif index == 0:
            node.next = self.head
            self.head = node
        # 在尾部插入
        # 1. 将尾结点的next指向该节点
        # 2. 将尾结点的引用指向该节点
        elif index == self.size:
            self.tail.next = node
            self.tail = node
        # 在中间插入
        # 1. 找到index的前一个节点preNode
        # 2. 将该节点的next指向preNode的下一个节点，即preNode.next
        # 3. 将preNode的next指向该节点
        # 注意：第2跟第3不能颠倒
        else:
            preNode = self.get(index-1)
            node.next = preNode.next
            preNode.next = node
        self.size = self.size + 1

    def remove(self, index):
        if self.size == 0:
            return None
        if index < 0 or index >= self.size:
            raise Exception('超出索引范围')
        if index == 0:
            removed_node = self.head
            self.head = self.head.next
        elif index == self.size - 1:
            preNode = self.get(index-1)
            removed_node = preNode.next
            preNode.next = None
            self.tail = preNode
        else:
            preNode = self.get(index-1)
            removed_node = preNode.next
            preNode.next = preNode.next.next
        self.size = self.size - 1
        return removed_node

    def printList(self):
        p = self.head
        while p:
            print(p.data)
            p = p.next

```

测试及输出

```python
list = ListNode()
list.insert(1, 0)
list.insert(2, 0)
list.insert(3, 2)
list.printList()  # 2 1 3
print('---------')
print(list.remove(0).data)  # 2
print(list.head.data)  # 1
print('=======')
list.printList()  # 1 3
print('=======')
print(list.tail.data)  # 3
print(list.get(1).data)  # 3
```

get与remove函数中传入的index的范围只能是[0, size)，但insert函数中传入的index范围却是[0, size]。因为insert的时候我们实际上是先申请了一个节点的空间，然后再去改变节点所存的值。而get与remove是只是访问链表的节点，没有申请空间的操作。