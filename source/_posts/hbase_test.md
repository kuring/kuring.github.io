---
title: HBase性能测试
Status: public
url: hbase_test
tags: Hadoop HBase
date: 2013-06-26
---

本文选择四台机器作为集群环境，hadoop采用0.20.2，HBase采用0.90.2，zookeeper采用独立安装的3.3.2稳定版。本文所采用的数据均为简单的测试数据，如果插入的数据量大可能会对结果产生影响。集群环境部署情况如下：

<table>
<tr>
    <td>机器名</td>
    <td>IP地址</td>
    <td>用途</td>
    <td>Hadoop模块</td>
    <td>HBase模块</td>
    <td>ZooKeeper模块</td>
</tr>
<tr>
    <td>server206</td>
    <td>192.168.20.6</td>
    <td>Master</td>
    <td>NameNode、JobTracker、SecondaryNameNode</td>
    <td>HMaster</td>
    <td>QuorumPeerMain</td>
</tr>
<tr>
    <td>ap1</td>
    <td>192.168.20.36</td>
    <td>Slave</td>
    <td>DataNode、TaskTracker</td>
    <td>HRegionServer</td>
    <td>QuorumPeerMain</td>
</tr>
<tr>
    <td>ap2</td>
    <td>192.168.20.38</td>
    <td>Slave</td>
    <td>DataNode、TaskTracker</td>
    <td>HRegionServer</td>
    <td>QuorumPeerMain</td>
</tr>
<tr>
    <td>ap2</td>
    <td>192.168.20.8</td>
    <td>Slave</td>
    <td>DataNode、TaskTracker</td>
    <td>HRegionServer</td>
    <td>QuorumPeerMain</td>
</tr>
</table>

# 单线程插入100万行
```
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;


public class InsertRowThreadTest {
	
	private static Configuration conf = null;
	
	private static String tableName = "blog";
	
	static {
		Configuration conf1 = new Configuration();
		conf1.set("hbase.zookeeper.quorum", "server206,ap1,ap2");
		conf1.set("hbase.zookeeper.property.clientPort", "2181");
        conf = HBaseConfiguration.create(conf1);
	}

	/**
	 * @param args
	 * @throws Exception 
	 */
	public static void main(String[] args) throws Exception {
        // 列族
        String[] familys = {"article", "author"};
        // 创建表
        try {
        	HBaseAdmin admin = new HBaseAdmin(conf);
        	if (admin.tableExists(tableName)) {
                System.out.println("表已经存在，首先删除表");
                admin.disableTable(tableName);
                admin.deleteTable(tableName);
            }
        	
            HTableDescriptor tableDesc = new HTableDescriptor(tableName);
            for(int i=0; i<familys.length; i++){
            	HColumnDescriptor columnDescriptor = new HColumnDescriptor(familys[i]);
                tableDesc.addFamily(columnDescriptor);
            }
            admin.createTable(tableDesc);
            System.out.println("创建表成功");
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		// 向表中插入数据
		long time1 = System.currentTimeMillis();
		System.out.println("开始向表中插入数据，当前时间为:" + time1);
		
		for (int i=0; i<1; i++) {
			InsertThread thread = new InsertThread(i * 1000000, 1000000, "thread" + i, time1);
			thread.start();
		}
	}
	
	public static class InsertThread extends Thread {
		
		private int beginSite;
		
		private int insertCount;
				
		private String name;
		
		private long beginTime;
		
		public InsertThread(int beginSite, int insertCount, String name, long beginTime) {
			this.beginSite = beginSite;
			this.insertCount = insertCount;
			this.name = name;
			this.beginTime = beginTime;
		}
		
		@Override
		public void run() {
			HTable table = null;
			try {
				table = new HTable(conf, Bytes.toBytes(tableName));
				table.setAutoFlush(false);
				table.setWriteBufferSize(1 * 1024 * 1024);
			} catch (IOException e1) {
				e1.printStackTrace();
			}
			
			System.out.println("线程" + name + "从" + beginSite + "开始插入");
			
			List<Put> putList = new ArrayList<Put>();
			for (int i=beginSite; i<beginSite + insertCount; i++) {
				Put put = new Put(Bytes.toBytes("" + i));
				put.add(Bytes.toBytes("article"), Bytes.toBytes("tag"), Bytes.toBytes("hadoop"));
				putList.add(put);
				if (putList.size() > 10000) {
					try {
						table.put(putList);
						table.flushCommits();
					} catch (IOException e) {
						e.printStackTrace();
					}
					putList.clear();
					try {
						Thread.sleep(5);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			try {
				table.put(putList);
				table.flushCommits();
				table.close();
			} catch (IOException e) {
				System.out.println("线程" + name + "失败");
				e.printStackTrace();
			}
			
			long currentTime = System.currentTimeMillis();
			System.out.println("线程" + name + "结束，用时" + (currentTime - beginTime));
		}
	}
}
```
测试5次的结果分布图如下：
![Image Title](/ref/hadoop/hbase_test_1.png)
其中Y轴单位为毫秒。平均速度在1秒插入3万行记录。

# 10个线程每个线程插入10万行
```
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;


public class InsertRowThreadTest {
	
	private static Configuration conf = null;
	
	private static String tableName = "blog";
	
	static {
		Configuration conf1 = new Configuration();
		conf1.set("hbase.zookeeper.quorum", "server206,ap1,ap2");
		conf1.set("hbase.zookeeper.property.clientPort", "2181");
        conf = HBaseConfiguration.create(conf1);
	}

	/**
	 * @param args
	 * @throws Exception 
	 */
	public static void main(String[] args) throws Exception {
        // 列族
        String[] familys = {"article", "author"};
        // 创建表
        try {
        	HBaseAdmin admin = new HBaseAdmin(conf);
        	if (admin.tableExists(tableName)) {
                System.out.println("表已经存在，首先删除表");
                admin.disableTable(tableName);
                admin.deleteTable(tableName);
            }
        	
            HTableDescriptor tableDesc = new HTableDescriptor(tableName);
            for(int i=0; i<familys.length; i++){
            	HColumnDescriptor columnDescriptor = new HColumnDescriptor(familys[i]);
                tableDesc.addFamily(columnDescriptor);
            }
            admin.createTable(tableDesc);
            System.out.println("创建表成功");
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		// 向表中插入数据
		long time1 = System.currentTimeMillis();
		System.out.println("开始向表中插入数据，当前时间为:" + time1);
		
		for (int i=0; i<10; i++) {
			InsertThread thread = new InsertThread(i * 100000, 100000, "thread" + i, time1);
			thread.start();
		}
	}
	
	public static class InsertThread extends Thread {
		
		private int beginSite;
		
		private int insertCount;
				
		private String name;
		
		private long beginTime;
		
		public InsertThread(int beginSite, int insertCount, String name, long beginTime) {
			this.beginSite = beginSite;
			this.insertCount = insertCount;
			this.name = name;
			this.beginTime = beginTime;
		}
		
		@Override
		public void run() {
			HTable table = null;
			try {
				table = new HTable(conf, Bytes.toBytes(tableName));
				table.setAutoFlush(false);
				table.setWriteBufferSize(1 * 1024 * 1024);
			} catch (IOException e1) {
				e1.printStackTrace();
			}
			
			System.out.println("线程" + name + "从" + beginSite + "开始插入");
			
			List<Put> putList = new ArrayList<Put>();
			for (int i=beginSite; i<beginSite + insertCount; i++) {
				Put put = new Put(Bytes.toBytes("" + i));
				put.add(Bytes.toBytes("article"), Bytes.toBytes("tag"), Bytes.toBytes("hadoop"));
				putList.add(put);
				if (putList.size() > 10000) {
					try {
						table.put(putList);
						table.flushCommits();
					} catch (IOException e) {
						e.printStackTrace();
					}
					putList.clear();
					try {
						Thread.sleep(5);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			try {
				table.put(putList);
				table.flushCommits();
				table.close();
			} catch (IOException e) {
				System.out.println("线程" + name + "失败");
				e.printStackTrace();
			}
			
			long currentTime = System.currentTimeMillis();
			System.out.println("线程" + name + "结束，用时" + (currentTime - beginTime));
		}
	}
}
```
耗时分布图为：
![Image Title](/ref/hadoop/hbase_test_2.png)
结果比单线程插入有提升。

# 20个线程每个线程插入5万行
```
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;


public class InsertRowThreadTest {
	
	private static Configuration conf = null;
	
	private static String tableName = "blog";
	
	static {
		Configuration conf1 = new Configuration();
		conf1.set("hbase.zookeeper.quorum", "server206,ap1,ap2");
		conf1.set("hbase.zookeeper.property.clientPort", "2181");
        conf = HBaseConfiguration.create(conf1);
	}

	/**
	 * @param args
	 * @throws Exception 
	 */
	public static void main(String[] args) throws Exception {
        // 列族
        String[] familys = {"article", "author"};
        // 创建表
        try {
        	HBaseAdmin admin = new HBaseAdmin(conf);
        	if (admin.tableExists(tableName)) {
                System.out.println("表已经存在，首先删除表");
                admin.disableTable(tableName);
                admin.deleteTable(tableName);
            }
        	
            HTableDescriptor tableDesc = new HTableDescriptor(tableName);
            for(int i=0; i<familys.length; i++){
            	HColumnDescriptor columnDescriptor = new HColumnDescriptor(familys[i]);
                tableDesc.addFamily(columnDescriptor);
            }
            admin.createTable(tableDesc);
            System.out.println("创建表成功");
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		// 向表中插入数据
		long time1 = System.currentTimeMillis();
		System.out.println("开始向表中插入数据，当前时间为:" + time1);
		
		for (int i=0; i<20; i++) {
			InsertThread thread = new InsertThread(i * 50000, 50000, "thread" + i, time1);
			thread.start();
		}
	}
	
	public static class InsertThread extends Thread {
		
		private int beginSite;
		
		private int insertCount;
				
		private String name;
		
		private long beginTime;
		
		public InsertThread(int beginSite, int insertCount, String name, long beginTime) {
			this.beginSite = beginSite;
			this.insertCount = insertCount;
			this.name = name;
			this.beginTime = beginTime;
		}
		
		@Override
		public void run() {
			HTable table = null;
			try {
				table = new HTable(conf, Bytes.toBytes(tableName));
				table.setAutoFlush(false);
				table.setWriteBufferSize(1 * 1024 * 1024);
			} catch (IOException e1) {
				e1.printStackTrace();
			}
			
			System.out.println("线程" + name + "从" + beginSite + "开始插入");
			
			List<Put> putList = new ArrayList<Put>();
			for (int i=beginSite; i<beginSite + insertCount; i++) {
				Put put = new Put(Bytes.toBytes("" + i));
				put.add(Bytes.toBytes("article"), Bytes.toBytes("tag"), Bytes.toBytes("hadoop"));
				putList.add(put);
				if (putList.size() > 10000) {
					try {
						table.put(putList);
						table.flushCommits();
					} catch (IOException e) {
						e.printStackTrace();
					}
					putList.clear();
					try {
						Thread.sleep(5);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			try {
				table.put(putList);
				table.flushCommits();
				table.close();
			} catch (IOException e) {
				System.out.println("线程" + name + "失败");
				e.printStackTrace();
			}
			
			long currentTime = System.currentTimeMillis();
			System.out.println("线程" + name + "结束，用时" + (currentTime - beginTime));
		}
	}
}
```
结果如下：
![Image Title](/ref/hadoop/hbase_test_3.png)
执行结果跟10个线程效果差不多。

# 10个线程每个线程插入100万行
代码跟前面例子雷同，为节约篇幅未列出。
执行结果如下：
![Image Title](/ref/hadoop/hbase_test_4.png)

# 20个线程每个线程插入50万行
执行结果如下：
![Image Title](/ref/hadoop/hbase_test_5.png)

# 总结
* 多线程比单线程的插入效率有所提高，开10个线程与开20个线程的插入行效率差不多。
* 插入效率存在不稳定情况，通过折线图可以看出。

# 相关文章
[在Linux上搭建Hadoop集群环境](/post/hadoop_setup)
[在Linux上搭建HBase集群环境](/post/hbase_setup)