# List的循环性能

我们常用的List有两种，ArrayList和LinkedList，但由于内部存储结构的不同，使用不同的遍历方法引起的效果则是千差万别的。

List | 存储结构 | 特点
---|---|---
ArrayList  | 数组结构 | 可以根据下标直接取值
LinkedList | 链表结构 | 如果需要寻找某一个下标的数值必须从头遍历

我们在遍历List的时候可能会这样做:

``` java
public void loopList(List<Integer> lists) {
    for (int i=0; i< lists.size(); i++) {
        Integer integer = lists.get(i);
        // TODO 处理数据
    }
}
```

这种做法很直观，乍一看并没有什么问题，但是仔细分析一下就能知道，在这种情况下使用ArrayList与LinkedList性能是完全不同的。

List | 循环时间复杂度 | get(i)时间复杂度 | 总时间复杂度
---|:---:|:---:|:---:
ArrayList  | O(n) | O(1) | O(n)
LinkedList | O(n) | O(n) | O(n<sup>2</sup>)

从时间复杂度上两者就直接差了一个数量级，可能这样说不明显，为了直观的表示，下面用代码测试处理10000条数据效率:

测试代码: 
```
ArrayList<Integer> arrayList = new ArrayList<>();
        LinkedList<Integer> linkedList = new LinkedList<>();
        for (int i=0; i<10000; i++){
            arrayList.add(i);
            linkedList.add(i);
        }

        long startTime1=System.currentTimeMillis();   //获取开始时间
        loopList(arrayList);
        long endTime1=System.currentTimeMillis(); //获取结束时间
        Log.i(TAG, "ArrayList： "+(endTime1-startTime1)+"ms");

        long startTime2=System.currentTimeMillis();   //获取开始时间
        loopList(linkedList);
        long endTime2=System.currentTimeMillis(); //获取结束时间
        Log.i(TAG, "LinkedList："+(endTime2-startTime2)+"ms");
```

结果:

``` shell
com.gcssloop.alltest I/MainActivity: ArrayList： 9ms
com.gcssloop.alltest I/MainActivity: LinkedList：709ms
```


