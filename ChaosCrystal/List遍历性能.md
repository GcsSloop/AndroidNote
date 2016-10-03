# ArrayList与LinkedList遍历性能比较

用实例测试ArrayList与LinkedList遍历性能。

## 结构差别:

我们常用的List有两种，ArrayList和LinkedList，虽然两者都是LIst，但由于内部存储结构的不同，使用不同的遍历方法性能却是千差万别的。

| List       | 存储结构 | 特点                    |
| ---------- | ---- | --------------------- |
| ArrayList  | 数组结构 | 可以根据下标直接取值。           |
| LinkedList | 链表结构 | 如果需要寻找某一个下标的数值必须从头遍历。 |



## 常见做法:

我们在遍历List的时候可能会这样做(从C语言上带来的习惯):

``` java
public void loopList(List<Integer> lists) {
    for (int i=0; i< lists.size(); i++) {
        Integer integer = lists.get(i);
        // TODO 处理数据
    }
}
```

这种做法很直观，乍一看并没有什么问题，但是仔细分析一下就能知道，在这种情况下使用ArrayList与LinkedList性能是完全不同的。

| List       | 循环时间复杂度 | get(i)时间复杂度 |      总时间复杂度      |
| ---------- | :-----: | :---------: | :--------------: |
| ArrayList  |  O(n)   |    O(1)     |       O(n)       |
| LinkedList |  O(n)   |    O(n)     | O(n<sup>2</sup>) |

从时间复杂度上两者就直接差了一个数量级，可能这样说不明显，为了直观的表示，下面用代码测试处理10000条数据效率:

### 性能测试:

创建数据:

```java
ArrayList<Integer> arrayList = new ArrayList<>();
LinkedList<Integer> linkedList = new LinkedList<>();
for (int i=0; i<10000; i++){
    arrayList.add(i);
    linkedList.add(i);
}
```

测试For循环用时:

``` java
long startTime1=System.currentTimeMillis();   //获取开始时间
loopList(arrayList);
iteratorList(arrayList);
long endTime1=System.currentTimeMillis(); //获取结束时间
Log.i("Time", "For-ArrayList： "+(endTime1-startTime1)+"ms");

long startTime2=System.currentTimeMillis();   //获取开始时间
loopList(linkedList);
long endTime2=System.currentTimeMillis(); //获取结束时间
Log.i("Time", "For-LinkedList："+(endTime2-startTime2)+"ms");
```

结果:

``` shell
com.gcssloop.alltest I/Time: For-ArrayList： 20ms
com.gcssloop.alltest I/Time: For-LinkedList：648ms
```

可以看到，仅仅处理10000条数据两者所需时间简直不能比较，当数据量越来越大的时候，LinkedList必然会耗费更多时间。



## 处理办法:

我们在Java中有 迭代器(Iterator) 以及 ForEach 循环，可以用它们来替代掉这个原始的 for 循环。

### 迭代器(Iterator):

```java
public void iteratorList(List<Integer> lists){
    Iterator<Integer> it = lists.iterator();
    while (it.hasNext()){
        Integer integer = it.next();
        // TODO 处理数据
    }
}
```

测试代码与上面类似，就省略了。

结果:

```shell
com.gcssloop.alltest I/Time: Iterator-ArrayList： 4ms
com.gcssloop.alltest I/Time: Iterator-LinkedList：6ms
```

**可以看到，两者最终耗费时间差不多而且均有大幅度提升。**



### ForEach循环:

```java
public void foreachList(List<Integer> lists){
    for (Integer i : lists) {
        // TODO 处理数据
    }
}
```

性能:

```shell
com.gcssloop.alltest I/Time: ForEach-ArrayList： 5ms
com.gcssloop.alltest I/Time: ForEach-LinkedList：5ms
```

由于 ForEach 循环底层使用的也是迭代器，所以和迭代器性能类似。



## 总结:

**推荐使用 迭代器(Iterator) 和 ForEach 遍历 List，不要使用传统的 For 循环。**

以下是测试处理10000条数据性能:

> 数值仅供参考，与运行环境相关，每次测试结果都可能稍有差别。

|    迭代方式    | ArrayList | LinkedList |
| :--------: | :-------: | :--------: |
|   For循环    |   20ms    |   648ms    |
|  Iterator  |    4ms    |    6ms     |
| for-each循环 |    5ms    |    5ms     |




## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>
