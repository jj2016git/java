# hashmap

## 存储结构

数组+链表

## 什么时候扩容？

数组被占用了capacity*factor时，扩容

链表长度超过一定值`HashMap#treeifyThreshold`，`put()/get()`效率降低

`treeifyThreshold=8` 有利于`put()/remove()`与`get()`效率平衡

- 解决方法：链表变形成红黑树（JDK8）

### `HashMap#put()`

hash是key的hashcode 高16位 ^低16位，防止key.hashCode()不分散

hash%16  === hash&15

100101000010100100010101010111010                 operand1

                                1111  &      operand2

---------------------------------------------------------------

                                1010

为了提升hashmap增删改查效率，必须保证`hash(key)&1111`尽量均衡

- operand1尽量均衡：

  hash%16  === hash&15key的hashcode 高16位 ^低16位，防止key.hashCode()不分散

- operand1&operand2尽量均衡：
  - 如果operand2有任意一位是0，就会导致operand1&operand2分散不均匀
  - 所以operand2必须是2^n-1, operand1&operand2范围为0~2^n-1, 数组大小必须为2^n，扩容必须要double

链表扩容后，计算在数组中的位置

100101000010100100010101010111010                 operand1

                                                                1111  &            operand2

------

                                                                 1010

- 需要计算第五位是0/1，hash & (2^n) == 0,不动，==1，移到新位置
- 扩容之后链表位置要么是原位置，要么是原位置+2^n

### hashmap并发问题

### 解决方案 ConcurrentHashmap

