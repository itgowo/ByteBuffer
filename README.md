# ByteBuffer
##### a simple binary data processing tool
##### 一个简单的二进制处理类

Github地址：https://github.com/itgowo/ByteBuffer

在TCP长连接中解决粘包半包有用到，项目[PackageMessage](https://www.jianshu.com/p/8a4a0ba2f54a)

### 故事：
写一个TCP长连接方案，遇到粘包分包问题，服务端于是用ByteArrayOutStream实现了，感觉太过于麻烦， 于是用java nio 的 ByteBuffer，但是不太灵活，最后用Netty的ByteBuf类，豁然开朗，尽然可以把代码压缩到这么少，转念一想，Android端怎么实现呢？毕竟引入Netty包太大，即使是部分代码也很大。Nio的ByteBuffer呢？遇见好用的自然看不上不太灵活的，于是写了此类解决。

### 一：功能介绍
Java Nio的ByteBuffer用起来不灵活（只说内存中二进制处理，MappedByteBuffer等不算）,功能上也只是比Java Nio的ByteBuffer多了几个，正好我想用的就是多的这几个，比如内部数组是支持动态扩展的，默认大小256，如果write超过容量，那么自动扩容到(data.length + addLength) * 2，所以不要存过大的数据。

Netty的CompositeByteBuf处理方式会把多个ByteBuf用list形式组织起来，每个ByteBuf都不会太大，这点我没有学，简单逻辑够用了，Netty那套很值得学习。

我写的这个ByteBuffer也支持销毁已读部分discardReadBytes()，获取原始array()和获取剩余可读数据readableBytesArray()，单独读写int、byte或者long，随意移动指针位置等。
从数组拷贝上使用System.arraycopy()，效率比较有保证

### 二：简单使用
###### ByteBuffer结构

|变量名|类型|说明|
|---|---|---|
|readerIndex|Integer |指针位置，即将读取的位置|
|writerIndex|Integer|指针位置，即将写入的位置|
|data|ByteArray|原始数据数组|

###### 创建ByteBuffer
```
    ByteBuffer buffer = ByteBuffer.newByteBuffer();
    ByteBuffer buffer = ByteBuffer.newByteBuffer(int capacity)
```
capacity参数指定初始数组大小，默认为***256***.

##### 方法

|方法名|参数一|参数二|返回值|说明|
|---|---|---|---|---|
|newByteBuffer()|||ByteBuffer|创建新ByteBuffer，默认大小256|
|newByteBuffer(int capacity)|默认大小||ByteBuffer|创建新ByteBuffer，大小为capacity|
|capacity()|||int|返回当前最大容量|
|array()|||ByteArray|获取原始数组，包含所有部分|
|clear()|||ByteBuffer|清理指针标记，状态为刚创建状态，但是data数据不变，新数组会覆盖旧数据，除了array()获取原始数组外无法得到旧数据|
|discardReadBytes()|||void|删除已读部分，重新初始化数组|
|readerIndex(int position)|重置指针位置||ByteBuffer|如果大于写入位置，则可读位置重置为写入位置，readableBytes()结果则为0|
|readerIndex()|||int|读指针位置|
|writerIndex()|||int|写指针位置|
|readableBytes()|||int|当前可读数据量，writerIndex - readerIndex|
|writableBytes()|||int|当前可写入数据量，每次触发扩容后都不一样|
|read()|||int|读取数据到byte，1 byte，从readIndex位置开始|
|readByte()|||byte|读取数据到byte，1 byte，从readIndex位置开始|
|readInt()|||int|读取integer值，读4 byte转换为integer，从readIndex位置开始|
|readBytes(byte[] bytes)|||int|读取数据到bytes，从readIndex位置开始|
|readBytes(ByteBuffer b)|待存入数据对象||int|读取数据到另一个ByteBuffer，读取数量|
|writeByte(byte b)|待写入数据||ByteBuffer|写入Byte数据，1 byte|
|write(int b)|待写入数据||ByteBuffer|写入int值的byte转换结果，即丢弃高位，1 byte|
|writeInt(int b)|待写入int值||ByteBuffer|写入integer数据，4 byte|
|writeBytes(byte[] b)|待写入ByteArray||ByteBuffer|写入数组|
|writeBytes(byte[] b, int dataLength)|待写入ByteArray|指定写入长度|ByteBuffer|写入数组,并指定写入长度|
|writeBytes(ByteBuffer b)|待写入ByteBuffer||ByteBuffer|写入一个ByteBuffer可读数据|
|writeBytes(ByteBuffer b, int dataLength)|待写入ByteBuffer|指定写入长度|ByteBuffer|写入一个ByteBuffer可读数据的部分长度|

### 三：原理解析

创建一个数组，通过读写index来表示当前数组可操作区域，不用多次创建新数组并拷贝了，虽然默认数组可能会变得很大，减少创建拷贝过程能提高性能，以空间换时间。另外数组一般不轻易超过4k吧，都是碎片的小数据，用这种方案最合适，如果是大数据量，那就没有什么意义了。
 
### 四：小期待
以下项目都是我围绕远程控制写的子项目。都给star一遍吧。😍

|项目(Github)|语言|其他地址|运行环境|项目说明|
|---|---|---|---|---|
|[PackageMessage](https://github.com/itgowo/PackageMessage)|Java|[简书](https://www.jianshu.com/p/8a4a0ba2f54a)|运行Java的设备|TCP粘包与半包解决方案|
|[ByteBuffer](https://github.com/itgowo/ByteBuffer)|Java|[简书](https://www.jianshu.com/p/ba68224f30e4)|运行Java的设备|二进制处理工具类|
|[RemoteDataControllerForAndroid](https://github.com/itgowo/RemoteDataControllerForAndroid)|Java|[简书](https://www.jianshu.com/p/eb692f5709e3)|Android设备|远程数据调试Android端|
|[RemoteDataControllerForWeb](https://github.com/itgowo/RemoteDataControllerForWeb)|JavaScript|[简书](https://www.jianshu.com/p/75747ff4667f)|浏览器|远程数据调试控制台Web端|
|[RemoteDataControllerForServer](https://github.com/itgowo/RemoteDataControllerForServer)|Java|[简书](https://www.jianshu.com/p/3858c7e26a98)|运行Java的设备|远程数据调试Server端|
|[MiniHttpClient](https://github.com/itgowo/MiniHttpClient)|Java|[简书](https://www.jianshu.com/p/41b0917271d3)|运行Java的设备|精简的HttpClient|
|[MiniHttpServer](https://github.com/itgowo/MiniHttpServer)|Java|[简书](https://www.jianshu.com/p/de98fa07140d)|运行Java的设备|支持部分Http协议的Server|
|[DataTables.AltEditor](https://github.com/itgowo/DataTables.AltEditor)|JavaScript|[简书](https://www.jianshu.com/p/a28d5a4c333b)|浏览器|Web端表格编辑组件|

[我的小站：IT狗窝](http://itgowo.com)
技术联系QQ:1264957104