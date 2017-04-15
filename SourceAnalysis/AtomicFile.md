# AtomicFile 源码解析

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1faa9dj0nf7j30rs05kaaf.jpg)

## 什么是 AtomicFile ?

AtomicFile 在 `android.support.v4.util` 包下，**是一个与文件相关的工具类，其作用是保证文件读写的原子性。** 即文件读写的时候全部成功才会更新文件，如果失败则不会影响文件内容。

看官方对其说明：

> Static library support version of the framework's `AtomicFile`, a helper class for performing atomic operations on a file by creating a backup file until a write has successfully completed.
>
> 静态支持库版本的AtomicFile，一个帮助类，用于通过创建备份文件对文件执行原子操作，直到写入成功完成。
>
> Atomic file guarantees file integrity by ensuring that a file has been completely written and sync'd to disk before removing its backup. As long as the backup file exists, the original file is considered to be invalid (left over from a previous attempt to write the file).
>
> 原子文件通过确保文件在删除其备份之前已经完全写入并同步到磁盘，从而保证文件的完整性。只要备份文件存在，原始文件将被视为无效（会尝试写入备份文件中）。
>
> Atomic file does not confer any file locking semantics. Do not use this class when the file may be accessed or modified concurrently by multiple threads or processes. The caller is responsible for ensuring appropriate mutual exclusion invariants whenever it accesses the file.
>
> 原子文件不提供任何文件锁定语义。当文件可能被多个线程或进程并发访问或修改时，不要使用此类。每当访问文件时，调用者都负责确保适当的互斥变量。

## 主要方法

|         返回值          | 方法名称和简介                                  |
| :------------------: | ---------------------------------------- |
|                      | **AtomicFile(File baseName)** <br/> 构造方法，通过File创建AtomicFile |
| **FileOutputStream** | **startWrite()** <br/> 准备写文件，获得一个向文件写入的字节流(FileOutputStream)。 |
|       **void**       | **finishWrite(FileOutputStream str)** <br/> 写入完毕时调用。 |
|       **void**       | **failWrite(FileOutputStream str)** <br/> 写入失败时调用。 |
| **FileInputStream**  | **openRead()** <br/> 准备读取文件，获得一个从文件读取的字节流(FileInputStream)。 |
|      **byte[]**      | **readFully()** <br/> 将文件中所有内容读取出来用 byte 数组返回，相当于更方便的 openRead 方法。 |
|       **File**       | **getBaseFile()** <br/> 获得原文件地址。         |
|       **void**       | **delete()** <br/> 删除 AtomicFile，包括原文件和备份文件。 |

## AtomicFile 原理

AtomicFile 原理其实非常简单，下面用一张图简单展示一下其原理：

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1faaeg4kqo5j30lu0tegp5.jpg)

## AtomicFile 源码解析

源码解析非常简单，根据上面表格中的内容按顺序一个一个看吧。



#### 构造函数：

根据传进来的文件创建 AtomicFile 类，同时创建辅助备份文件(空文件)。

```java
private final File mBaseName;
private final File mBackupName;

public AtomicFile(File baseName) {
    mBaseName = baseName;                                 // 记录原始文件
    mBackupName = new File(baseName.getPath() + ".bak");  // 创建备份文件
}
```



#### 开始写入(startWrite)

在获取向文件写入的输出流之前，它会对原文件进行备份，同时清除原文件内容。

同时根据其原理可知，它并不是线程安全的，如果需要多线程操作等，最好自己加锁，如果一个线程未写完，直接开启了另一个线程进行写入可能会导致文件内容丢失。

**另外使用这种方法获得到的 FileOutputStream 不要直接关闭，写入完成的时候需要调用 `finishWrite` 或者`failWrite` 进行关闭，否则下次读取的时候会因为备份文件存在而使本次写入失效。**

```java
public FileOutputStream startWrite() throws IOException {
    // 当原文件存在，备份文件不存在的时候，原文件更名为备份文件
    if (mBaseName.exists()) {
        if (!mBackupName.exists()) {
          	// 如果原文件存在且备份文件不存在，直接将原文件重命名为备份文件
            if (!mBaseName.renameTo(mBackupName)) {
                Log.w("AtomicFile", "Couldn't rename file " + mBaseName
                        + " to backup file " + mBackupName);
            }
        } else {
            mBaseName.delete();	// 删除原文件
        }
    }
  	// 保证原文件存在，并根据原文件创建一个输出流。
    FileOutputStream str = null;
    try {
        str = new FileOutputStream(mBaseName);
    } catch (FileNotFoundException e) {
        File parent = mBaseName.getParentFile();
        if (!parent.mkdirs()) {
            throw new IOException("Couldn't create directory " + mBaseName);
        }
        FileUtils.setPermissions(
            parent.getPath(),
            FileUtils.S_IRWXU|FileUtils.S_IRWXG|FileUtils.S_IXOTH,
            -1, -1);
        try {
            str = new FileOutputStream(mBaseName);
        } catch (FileNotFoundException e2) {
            throw new IOException("Couldn't create " + mBaseName);
        }
    }
    return str;	// 返回输出流。
}
```



#### 写入完成(finishWrite)

写入完成的时候需要用户调用该方法，该方法会关闭 FileOutputStream ，并且会删除备份文件以保证文件的唯一性。

```java
public void finishWrite(FileOutputStream str) {
    if (str != null) {
        FileUtils.sync(str);
        try {
            str.close();
            mBackupName.delete();
        } catch (IOException e) {
            Log.w("AtomicFile", "finishWrite: Got exception:", e);
        }
    }
}
```



#### 写入失败(failWrite)

写入失败时会调用该方法，该方法会关闭 FileOutputStream，并且从备份文件恢复文件内容。

```java
public void failWrite(FileOutputStream str) {
    if (str != null) {
        FileUtils.sync(str);
        try {
            str.close();
            mBaseName.delete();
            mBackupName.renameTo(mBaseName);
        } catch (IOException e) {
            Log.w("AtomicFile", "failWrite: Got exception:", e);
        }
    }
}
```



#### 准备读取(openRead)

该方法会获取到一个读取文件的输入流(FileInputStream)，并且在备份文件存在时，会简单粗暴的删除原文件，并从备份文件恢复内容。

```java
public FileInputStream openRead() throws FileNotFoundException {
    if (mBackupName.exists()) {
        mBaseName.delete();
        mBackupName.renameTo(mBaseName);
    }
    return new FileInputStream(mBaseName);
}
```



#### 读取全部(readFully)

该方法会将文件中的所有内容转化为一个 byte 数组并且返回，所以不建议使用该方法读取大文件。

```java
public byte[] readFully() throws IOException {
    FileInputStream stream = openRead();
    try {
        int pos = 0;
        int avail = stream.available();
        byte[] data = new byte[avail];
      	// 通过循环读取的方式将文件所有内容都装入 byte 数组中。
        while (true) {
            int amt = stream.read(data, pos, data.length-pos);
            //Log.i("foo", "Read " + amt + " bytes at " + pos
            //        + " of avail " + data.length);
            if (amt <= 0) {
                //Log.i("foo", "**** FINISHED READING: pos=" + pos
                //        + " len=" + data.length);
                return data;
            }
            pos += amt;
            avail = stream.available();
            if (avail > data.length-pos) {
                byte[] newData = new byte[pos+avail];
                System.arraycopy(data, 0, newData, 0, pos);
                data = newData;
            }
        }
    } finally {
        stream.close();
    }
}
```



#### 获取原文件(getBaseFile)

该方法用于获取原文件路径，但通常情况下并不推荐使用，因为获取到的原文件可能是损坏的或者无效的。

```java
public File getBaseFile() {
    return mBaseName;
}
```



#### 删除(delete)

该方法会直接删除原文件和备份文件。

```java
public void delete() {
    mBaseName.delete();
    mBackupName.delete();
}
```



## 简单示例



## FAQ

**Q：AtmoicFile 适用于哪些文件？**

> A：**AtomicFile 适用一次性写入的文件**。根据 AtomicFile 原理可知，每一次获取写入文件的输出流的时候都会清空原文件的内容，所以是无法给文件追加内容的。

**Q：AtmoicFile 是否是线程安全的？**

> A：根据其原理就可知它没有带线程锁，所以**AtomicFile并不能保证线程安全**。

**Q：文件写入完毕后可以直接关闭输出字节流(FileOutputStream)么？**

> A：AtomicFile写入完毕后是不允许直接关闭字节流的，因为直接关闭字节流会导致备份文件没有删除，因而下次读取的时候会导致读取到的是原文件，而不是更新后的文件。  
> **写入完毕后应调用 finishWrite(正常写入完成) 或者 failWrite(写入失败)。**

**Q：文件读取完毕后可以直接关闭输入字节流(FileInputStream)么？**

> A：可以。



## 结语

AtomicFile 作为一个工具类，有其方便之处，同时也有一些需要注意的地方，例如不能追加内容，另外，AtomicFile 除了上述方法之外，还有另外几个方法没有在本文中说明，这几个方法由于不安全，已经被标注删除和隐藏，故本文就不在赘述了。

事实上 AtomicFile 逻辑非常简单，应用场景也有限，如果需要的话自己徒手写一个也是可以的，个人觉得它的作用并不太大，如果以后大家在源码或者其他地方见到了这个类，知道它是干什么的就行了。

**最后，任何工具都是双刃剑，用好了伤人，用不好伤己，希望大家用之前好好了解一下其利弊，权衡之后再做决定。**



## 参考资料

[Guide - AtomicFile](https://developer.android.com/reference/android/support/v4/util/AtomicFile.html)  
[源码 - AtomicFile](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/util/AtomicFile.java)



## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>