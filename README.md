# hbase-2.1.0-cdh6.1.1
解决无法访问远程内网HBase的问题  

# redirect master
HBase读写过程无需master参与，但是客户端在连接集群时，zookeeper会先判断master是否存活，所以如果master连接不上的话zookeeper这一关就过不了，会提示"zookeeper available but no active master location found".
这一步的修改：  
  1、ConnectionImplementation.java的内部类MasterServiceStubMaker.java中，makeStubNoRetries()方法中，将根据masterAddress得到的ServerName对象进行修改，将返回的master端口号修改为映射后的端口号，然后把新的ServerName对象sn替换原有的。  
这一步做完后，可以建表但无法写入。


# redirect RegionServers
HBase的读写需要先拿到RS的实际地址，这里不用再管meta表在哪个RS，只需要关心如何将从meta表中拿到的regioninfo中的信息进行修改
修改点依然是ConnectionImplementation.java，cacheLocation()方法，该方法是缓存新发现的HRegionLocation，在此处就把端口号全部替换

# 需要的配置项
ACTUAL_MASTER=master,16000
EXPECT_MASTER=master,110
SEPERATOR=/
ACTUAL_RS=slave1,16020/slave2,16020
EXPECT_RS=slave1,111/slave2,112

上述修改完成后可以满足需求：远程建表以及远程写入，至于是否对其他功能有影响，或者是否可以远程使用其他功能，尚未进行测试。
