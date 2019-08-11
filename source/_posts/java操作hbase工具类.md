---
title: java操作hbase工具类
date: 2018-03-29 19:23:22
tags: HBase
categories: HBase
---
HBase是一个基于HDFS的数据库，拥有高可用、大量数据存储、列式存储等特点，在非结构化数据与半结构化数据存储方面，有很大的优势。我们一般测试时使用hbase shell命令行的方式来操作hbase数据库比较方便，但是在数据逻辑处理比较复杂时，那肯定是用它提供的API来操作更方便啦，下面就来给出一个java版操作hbase的工具类，提供给大家，我自己也一直使用这个类。
<!-- more -->
备注：本工具类使用的环境：hbase1.4.1	jdk1.8		hadoop3.0
# maven项目添加依赖
```
<!--hadoop/hbase都要依赖(RPC通信)，注意protobuf-java的版本，hbase1.4.1自带的protobuf-java版本是2.5.0的，所以如果你的程序是跑在服务器上的，需要跟服务器一致，不然会出现NoClsssFoundError-->
<!--https://mvnrepository.com/artifact/com.google.protobuf/protobuf-java-->
<dependency>
	<groupId>com.google.protobuf</groupId>
	<artifactId>protobuf-java</artifactId>
	<version>3.5.1</version>
</dependency>
<!--hbase-->
<dependency>
	<groupId>org.apache.zookeeper</groupId>
	<artifactId>zookeeper</artifactId>
	<version>${zookeeper.version}</version>
</dependency>
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-client</artifactId>
	<version>${hbase.version}</version>
</dependency>
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-common</artifactId>
	<version>${hbase.version}</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase -->
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase</artifactId>
	<version>1.4.1</version>
	<type>pom</type>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-spark -->
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-spark</artifactId>
	<version>1.2.0-cdh5.14.0</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-server -->
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-server</artifactId>
	<version>1.4.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/fastutil/fastutil 这里使用fastutil，对比javautil自带集合类，它的读写性能更优，尤其在大数据的情况下，所以当你写的mr或者spark程序，使用到fastutil，会提升一些性能-->
<dependency>
	<groupId>fastutil</groupId>
	<artifactId>fastutil</artifactId>
	<version>5.0.5</version>
</dependency>
```
# 代码
```
package com.payegis.czl.util;

import it.unimi.dsi.fastutil.objects.ObjectArrayList;
import net.sf.json.JSONObject;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.*;
import org.apache.hadoop.hbase.io.compress.Compression;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.log4j.Logger;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

/**
 * User: chenzuoli
 * Date: 2018/3/29
 * Time: 9:20
 * Description: Java操作HBase工具类
 * Ps: Java HBase
 */

public class HBaseUtil {
    public static Configuration conf;
    public static Connection connection;
    public static Admin admin;
    public static Table table;
    private static Logger logger = Logger.getLogger(HBaseUtil.class);

    static {
        try {
            conf = HBaseConfiguration.create();
            conf.set("hbase.zookeeper.property.clientPort", "2181");
            conf.set("hbase.zookeeper.quorum", "dev11,dev13,dev14");
            connection = ConnectionFactory.createConnection(conf);
            admin = connection.getAdmin();
            logger.info("初始化hbase连接成功！");
        } catch (IOException e) {
            logger.error("初始化hbase连接异常！");
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("初始化hbase连接异常！");
            e.printStackTrace();
        }
    }

    /**
     * @Description: 建表，如果表存在，那么不创建。如果未指定列族名称，默认定义一个cf1
     * @Param: [tableName, familyName]
     * @return: boolean
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 9:24
     */
    public static boolean createTable(String tableName, String familyName) {
        boolean flag = false;
        if (familyName == null || familyName.length() == 0) {
            familyName = "cf1";
        }
        TableName tbl = TableName.valueOf(tableName);
        Admin admin = null;
        try {
            admin = connection.getAdmin();
            if (admin.tableExists(tbl)) {
                logger.info("Table " + tbl.getNameAsString() + " is already exists!");
                return flag;
            }
            HTableDescriptor tableDescriptor = new HTableDescriptor(tbl);
            tableDescriptor.addFamily(new HColumnDescriptor(familyName).setCompressionType(Compression.Algorithm.SNAPPY));
            admin.createTable(tableDescriptor);
            logger.info("Create table " + tbl.getNameAsString() + " success!");
            flag = true;
        } catch (IOException e) {
            logger.error("Create table failed!");
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("Create table failed!");
            e.printStackTrace();
        }
        return flag;
    }

    /**
     * @Description: 插入一条数据到hbase
     * @Param: [connection, tableName, rowkey, columnFamily, key, value]
     * @return: void
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/28
     * @Time: 14:06
     */
    public static void insertOne(String tableName, String rowkey, String columnFamily, String key, String value) {
        Table table = null;
        try {
            table = connection.getTable(TableName.valueOf(tableName));
            Put put = new Put(Bytes.toBytes(rowkey));
            put.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(key), Bytes.toBytes(value));
            table.put(put);
        } catch (IOException e) {
            logger.error("insert hbase failed: " + rowkey + "," + columnFamily + "," + key + "," + value);
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("insert hbase failed: " + rowkey + "," + columnFamily + "," + key + "," + value);
            e.printStackTrace();
        } finally {
            closeTableAndResult(table, null);
        }
    }

    /**
     * @Description: 批量插入数据到hbase
     * @Param: [filePath, tableName, familyName]
     * @return: void
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/30
     * @Time: 13:31
     */
    public static void insertBatch(String filePath, String tableName, String familyName) {
        ObjectArrayList<Put> puts = new ObjectArrayList<>();
        Table table = null;
        FileInputStream fis = null;
        BufferedReader br = null;
        try {
            table = connection.getTable(TableName.valueOf(tableName));
            fis = new FileInputStream(filePath);
            br = new BufferedReader(new InputStreamReader(fis));
            String line = br.readLine();
            while (line != null) {
                JSONObject lineJsonObject = JSONObject.fromObject(line);
                String rowkey = MD5Utils.strToMd5_16(UUID.randomUUID().toString());
                Set<String> keys = lineJsonObject.keySet();
                Put put = new Put(Bytes.toBytes(rowkey));
                for (String key : keys) {
                    put.addColumn(Bytes.toBytes(familyName), Bytes.toBytes(key), Bytes.toBytes(lineJsonObject.optString(key)));
                    puts.add(put);
                }
                line = br.readLine();
            }
            table.put(puts);
        } catch (IOException e) {
            logger.error("insert batch data to hbase failed!");
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("insert batch data to hbase failed!");
            e.printStackTrace();
        } finally {
            try {
                if (table != null) table.close();
                if (fis != null) fis.close();
                if (br != null) br.close();
            } catch (IOException e) {
                logger.error("close table or stream failed!");
                e.printStackTrace();
            }
        }
    }

    /**
     * @Description: 批量插入数据到hbase
     * @Param: [rows, tableName, familyName]
     * @return: void
     * @Author: CHEN ZUOLI
     * @Date: 2018/4/2
     * @Time: 10:19
     */
    public static void insertBatch(List<Map<String, Object>> rows, String tableName, String familyName) {
        ObjectArrayList<Put> puts = new ObjectArrayList<>();
        Table table = null;
        try {
            table = connection.getTable(TableName.valueOf(tableName));
            for (Map<String, Object> row : rows) {
                String rowkey = MD5Utils.strToMd5_16(UUID.randomUUID().toString());
                Put put = new Put(Bytes.toBytes(rowkey));
                for (Map.Entry<String, Object> kv : row.entrySet()) {
                    String key = kv.getKey();
                    Object value = kv.getValue();
                    if (value == null) {
                        put.addColumn(Bytes.toBytes(familyName), Bytes.toBytes(key), null);
                    } else {
                        put.addColumn(Bytes.toBytes(familyName), Bytes.toBytes(key), Bytes.toBytes(value.toString()));
                    }
                }
                puts.add(put);
            }
            table.put(puts);
        } catch (IOException e) {
            logger.error("insert batch data to hbase failed!");
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("insert batch data to hbase failed!");
            e.printStackTrace();
        } finally {
            closeTableAndResult(table, null);
        }
    }

    /**
     * @Description: 删除一张表
     * @Param: [tableName]
     * @return: boolean
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 9:40
     */
    public static boolean dropTable(String tableName) {
        boolean flag = false;
        try {
            admin.disableTable(TableName.valueOf(tableName));
            admin.deleteTable(TableName.valueOf(tableName));
            flag = true;
        } catch (IOException e) {
            logger.error("delete " + tableName + " table failed!");
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("delete " + tableName + " table failed!");
            e.printStackTrace();
        }
        return flag;
    }

    /**
     * @Description: 根据rowkey删除一条记录
     * @Param: [tablename, rowkey]
     * @return: boolean
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 9:40
     */
    public static boolean deleteOneRowByRowkey(String tablename, String rowkey) {
        boolean flag = false;
        try {
            Delete d = new Delete(rowkey.getBytes());
            table.delete(d);
            logger.info("delete row " + rowkey + " success!");
            flag = true;
        } catch (IOException e) {
            logger.error("delete row " + rowkey + " failed!");
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("delete row " + rowkey + " failed!");
            e.printStackTrace();
        }
        return flag;
    }

    /**
     * @Description: 批量删除rowkey
     * @Param: [tablename, rowkeyList]
     * @return: boolean
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 9:47
     */
    public static boolean deleteBatchRowByRowkey(String tablename, List<String> rowkeyList) {
        boolean flag = false;
        ObjectArrayList<Delete> listDelete = new ObjectArrayList<>();
        try {
            for (int i = 0; i < rowkeyList.size(); i++) {
                Delete delete = new Delete(rowkeyList.get(i).getBytes());
                listDelete.add(delete);
            }
            table.delete(listDelete);
            logger.info("delete row list " + rowkeyList + " success!");
            flag = true;
        } catch (IOException e) {
            logger.error("delete row " + rowkeyList + " failed!");
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("delete row " + rowkeyList + " failed!");
            e.printStackTrace();
        }
        return flag;
    }

    /**
     * @Description: 查询表中所有数据
     * @Param: [tableName]
     * @return: List<HashMap<String,HashMap<String,String>>>
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 9:51
     */
    public static List<HashMap<String, HashMap<String, String>>> queryAll(String tableName) {
        ObjectArrayList<HashMap<String, HashMap<String, String>>> rowMapList = new ObjectArrayList<>(); // <familyName, <columnName, columnValue>>
        Table table = null;
        ResultScanner rs = null;
        try {
            table = connection.getTable(TableName.valueOf(tableName));
//            ResultScanner rs = table.getScanner(new Scan().setMaxVersions()); // 获取所有版本数据
            rs = table.getScanner(new Scan());
            for (Result r : rs) {
                rowMapList.add(resolveResult(r));
            }
        } catch (IOException e) {
            logger.error("Get all table data failed!");
            e.printStackTrace();
        } finally {
            closeTableAndResult(table, rs);
        }
        return rowMapList;
    }

    /**
     * @Description: 单条件查询, 根据rowkey查询唯一一条记录
     * @Param: [tableName, rowKey]
     * @return: List<HashMap<String,HashMap<String,String>>>
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 10:47
     */
    public static List<HashMap<String, HashMap<String, String>>> queryByCondition(String tableName, String rowKey) {
        ObjectArrayList<HashMap<String, HashMap<String, String>>> rowMapList = new ObjectArrayList<>(); // <familyName, <columnName, columnValue>>
        Table table = null;
        try {
            Get get = new Get(rowKey.getBytes());
//            get.setMaxVersions(); // 获取所有版本数据
            table = connection.getTable(TableName.valueOf(tableName));
            Result r = table.get(get);
            rowMapList.add(resolveResult(r));
            logger.info("获得到rowkey: " + new String(r.getRow()));
        } catch (IOException e) {
            logger.error("Get table one data failed!");
            e.printStackTrace();
        } finally {
            closeTableAndResult(table, null);
        }
        return rowMapList;
    }

    /**
     * @Description: 单条件按查询，查询多条记录
     * @Param: [tableName]
     * @return: List<HashMap<String,HashMap<String,String>>>
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 13:16
     */
    public static List<HashMap<String, HashMap<String, String>>> queryByCondition(String tableName, String familyName, String columnName, String columnValue) {
        ObjectArrayList<HashMap<String, HashMap<String, String>>> rowMapList = new ObjectArrayList<>(); // <familyName, <columnName, columnValue>>
        Table table = null;
        ResultScanner rs = null;
        try {
            table = connection.getTable(TableName.valueOf(tableName));
            Filter filter = new SingleColumnValueFilter(Bytes.toBytes(familyName), Bytes.toBytes(columnName), CompareFilter.CompareOp.EQUAL, Bytes.toBytes(columnValue)); // 当列columnName的值为columnValue时进行查询
            Scan s = new Scan();
            s.setFilter(filter);
            rs = table.getScanner(s);
            for (Result r : rs) {
                rowMapList.add(resolveResult(r));
            }
        } catch (Exception e) {
            logger.error("query with one filter failed!");
            e.printStackTrace();
        } finally {
            closeTableAndResult(table, rs);
        }
        return rowMapList;
    }

    /**
     * @Description: 组合条件查询
     * @Param: [tableName]
     * @return: List<HashMap<String,HashMap<String,String>>>
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 13:26
     */
    public static List<HashMap<String, HashMap<String, String>>> queryByCondition(String tableName, String familyName, HashMap<String, String> paramMap) {
        ObjectArrayList<HashMap<String, HashMap<String, String>>> rowMapList = new ObjectArrayList<>(); // <familyName, <columnName, columnValue>>
        Table table = null;
        ResultScanner rs = null;
        try {
            table = connection.getTable(TableName.valueOf(tableName));
            FilterList filterList = new FilterList();
            for (Map.Entry<String, String> entry : paramMap.entrySet()) {
                Filter filter = new SingleColumnValueFilter(Bytes.toBytes(familyName), Bytes.toBytes(entry.getKey()), CompareFilter.CompareOp.EQUAL, Bytes.toBytes(entry.getValue()));
                filterList.addFilter(filter);
            }
            Scan scan = new Scan();
            scan.setFilter(filterList);
            rs = table.getScanner(scan);
            for (Result r : rs) {
                rowMapList.add(resolveResult(r));
            }
            rs.close();
        } catch (Exception e) {
            logger.error("query with more filter failed!");
            e.printStackTrace();
        } finally {
            closeTableAndResult(table, rs);
        }
        return rowMapList;
    }

    /**
    * @Description: 查询hbase，匹配rowkey前缀为dianRong的行
    * @Param: [tableName]
    * @return: java.util.List<java.util.HashMap<java.lang.String,java.util.HashMap<java.lang.String,java.lang.String>>>
    * @Author: CHEN ZUOLI
    * @Date: 2018/4/3
    * @Time: 20:21
    */
    public static List<HashMap<String, HashMap<String, String>>> rowkeyFuzzyQuery(String tableName){
        ObjectArrayList<HashMap<String, HashMap<String, String>>> rowMapList = new ObjectArrayList<>();
        Table table = null;
        ResultScanner rs = null;
        try {
            table = connection.getTable(TableName.valueOf(tableName));
            Scan scan = new Scan();
            Filter filter = new RowFilter(CompareFilter.CompareOp.EQUAL, new RegexStringComparator("dianRong.*"));
            scan.setFilter(filter);
            rs = table.getScanner(scan);
            for (Result r : rs) {
                rowMapList.add(resolveResult(r));
            }
        } catch (Exception e) {
            logger.error("query with more filter failed!");
            e.printStackTrace();
        } finally {
            closeTableAndResult(table, rs);
        }
        return rowMapList;
    }

    /**
     * @Description: 解析查询hbase得到的结果，放入到HashMap中
     * @Param: [result]
     * @return: java.util.HashMap<String,HashMap<String,String>>
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 13:52
     */
    public static HashMap<String, HashMap<String, String>> resolveResult(Result result) {
        HashMap<String, HashMap<String, String>> rowMap = new HashMap<>(); // <familyName, <columnName, columnValue>>
        HashMap<String, String> kvMap = new HashMap<>();
        NavigableMap<byte[], NavigableMap<byte[], NavigableMap<Long, byte[]>>> map = result.getMap();
        for (Map.Entry<byte[], NavigableMap<byte[], NavigableMap<Long, byte[]>>> entry : map.entrySet()) {
            String familyName = new String(entry.getKey());
            NavigableMap<byte[], NavigableMap<Long, byte[]>> valueInfoMap = entry.getValue();
            for (Map.Entry<byte[], NavigableMap<Long, byte[]>> valueInfo : valueInfoMap.entrySet()) {
                String key = new String(valueInfo.getKey());
                NavigableMap<Long, byte[]> values = valueInfo.getValue();
                Map.Entry<Long, byte[]> firstEntry = values.firstEntry();
                Long timestampLastest = firstEntry.getKey();
                String valueLastest = new String(firstEntry.getValue());
                logger.info("familyName: " + familyName + ", key: " + key + ", value: " + valueLastest + ", timestamp: " + timestampLastest);
//                for (Map.Entry<Long, byte[]> vals : values.entrySet()) {
//                    Long timestamp = vals.getKey();
//                    String value = new String(vals.getValue());
//                }
                kvMap.put(key, valueLastest);
                rowMap.put(familyName, kvMap);
            }
        }
        return rowMap;
    }

    public static void closeTableAndResult(Table table, ResultScanner rs){
        try {
            if (rs != null) rs.close();
            if (table != null) table.close();
        } catch (IOException e) {
            logger.error("close table failed!");
            e.printStackTrace();
        }
    }

}
```
好了，文章到这里就结束了，如果大家在使用过程中，遇到什么问题，请联系我chenzuoli709@gmail.com。