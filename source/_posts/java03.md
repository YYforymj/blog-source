---
title: java基础03
date: 2017-04-01 14:38:36
categories: java
---
> 以下内容为传智博客的《java基础入门》一书的读书笔记

<!-- more -->

## 第七章
* 集合可以分为单列集合与多列集合
![Collection](\images\java03\Collection.png)
![Map](\images\java03\Map.png)

### 单列集合(Collection)
* List接口：允许出现重复的元素；
1. ArrayList：内部封装了长度可变数组，因此不适用大量增删改查操作，但查找效率高；
2. LinkedList：内部维护了双向循环链表，增删改查效率高；
3. Vector：用法与ArrayList相同，但Vector线程安全，ArrayList线程不安全；
* Set接口：元素不允许重复，且元素无序；
1. HashSet：存取查找性能良好，可以存放null值；当调用HashSet的add()时，首先调用存入对象的hashcode()方法，获得对象的hash值，然后计算存储位置并存储，如果该位置有元素则调用equals()比较，比较结果为false则存入HashSet，true说明有重复元素，舍弃。
2. TreeSet：采用自平衡二叉树存储数据

### 双列集合(Map<key,value>)
1. HashMap：key不可以重复，如果存储了相同key值的数据，后存储的数据会覆盖先存储的数据；HashMap迭代出的数据与存入顺序不一致
2. LinkedHashMap：内部使用双向链表，所以元素迭代顺序与存入顺序一致；
3. HashTable：与HashMap相似，但HashMap线程不安全，HashTable线程安全，但存取较慢；
4. Properties：HashTable的子类，通常用于配置信息的读写。

## 第八章
* 操作包含中文的纯文本使用字符流
* SequenceInputStream用于将几个流串联在一起合并为一个流，它会依次从所有被串联的流中读取数据；
* BufferWriter.newline()可以写入一个换行符；
* 选择排序：每一次循环过程中，通过比较选择出最大值，记下最大值的角标，利用角标将最大值与此轮最后一个位置的数据进行交换，然后以去掉这个最大值的序列为新序列重复前面步骤。例子：
数组排序前    9 8 3 5 2    
第一轮循环    2 8 3 5 9       
第二轮循环    2 5 3 8 9
第三轮循环    2 3 5 8 9