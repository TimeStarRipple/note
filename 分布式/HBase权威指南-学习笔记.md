#HBase权威指南-学习笔记
##内容提要
###大体概括
1. Hadoop将HBase的可伸缩性变得简单
2. 把大型数据集分布到商业服务器集群中
3. 网关服务器访问HBase

###细节说明
1. 了解HBase框架细节，包括存储格式，预写日志，后台进程
2. HBase集成了MapReduce框架，了解调节集群，设计模式，拷贝表，导入批量数据和删除节点的操作

数据存储，先进入到提交日志，然后，才会将这些数据写入内存中的memstore，最后再写入磁盘的HFile

```
  <property>
    <name>hbase.rootdir</name>
    <value>file:///~/hbase</value>
  </property>
```
![好的](http://MarkdownPhotos/Res/test.jpg)
