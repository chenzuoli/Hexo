---
title: java操作hbase工具类
date: 2018-03-29 19:23:22
tags: HBase
categories: HBase
---
HBase是一个基于HDFS的数据库，拥有高可用、大量数据存储、列式存储等特点，在非结构化数据与半结构化数据存储方面，有很大的优势。我们一般测试时使用hbase shell命令行的方式来操作hbase数据库比较方便，但是在数据逻辑处理比较复杂时，那肯定是用它提供的API来操作更方便啦，下面就来给出一个java版操作hbase的工具类，提供给大家，我自己也一直使用这个类。
<!-- more -->
备注：本工具类使用的环境：hbase1.4.1	jdk1.8		hadoop3.0
```
package com.payegis.czl.util;

import it.unimi.dsi.fastutil.objects.ObjectArrayList;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.CompareFilter;
import org.apache.hadoop.hbase.filter.Filter;
import org.apache.hadoop.hbase.filter.FilterList;
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
import org.apache.hadoop.hbase.io.compress.Compression;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.log4j.Logger;

import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.NavigableMap;

/**
 * User: 陈作立
 * Date: 2018/3/29
 * Time: 9:20
 * Description: Java操作HBase工具类
 * Ps: Java HBase
 */

public class HBaseUtil {
    private static Logger logger = Logger.getLogger(HBaseUtil.class);
    public static Admin admin;
    public static Table table;
    private static Configuration conf;
    private static Connection conn;

    static {
        try {
            conf = HBaseConfiguration.create();
            conf.set("hbase.zookeeper.property.clientPort", "2181");
            conf.set("hbase.zookeeper.quorum", "dev11,dev13,dev14");
            conn = ConnectionFactory.createConnection(conf);
            admin = conn.getAdmin();
            logger.info("初始化成功！");
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
     * @Param: []
     * @return: void
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
            admin = conn.getAdmin();
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
    public static void insertOne(Connection connection, String tableName, String rowkey, String columnFamily, String key, String value) {
        try {
            Table table = connection.getTable(TableName.valueOf(tableName));
            Put put = new Put(Bytes.toBytes(rowkey));
            put.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(key), Bytes.toBytes(value));
            table.put(put);
        } catch (IOException e) {
            logger.error("数据插入hbase失败：" + rowkey + "," + columnFamily + "," + key + "," + value);
            e.printStackTrace();
        } catch (Exception e) {
            logger.error("数据插入hbase失败：" + rowkey + "," + columnFamily + "," + key + "," + value);
            e.printStackTrace();
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
     * @return: void
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
     * @return: void
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 9:51
     */
    public static List<HashMap<String, HashMap<String, String>>> queryAll(String tableName) {
        ObjectArrayList<HashMap<String, HashMap<String, String>>> rowMapList = new ObjectArrayList<>(); // <familyName, <columnName, columnValue>>
        Table table = null;
        ResultScanner rs = null;
        try {
            table = conn.getTable(TableName.valueOf(tableName));
//            ResultScanner rs = table.getScanner(new Scan().setMaxVersions()); // 获取所有版本数据
            rs = table.getScanner(new Scan());
            for (Result r : rs) {
                rowMapList.add(resolveResult(r));
            }
        } catch (IOException e) {
            logger.error("Get all table data failed!");
            e.printStackTrace();
        } finally {
            try {
                if (rs != null) rs.close();
                if (table != null) table.close();
            } catch (IOException e) {
                logger.error("close table failed!");
                e.printStackTrace();
            }
        }
        return rowMapList;
    }

    /**
     * @Description: 单条件查询, 根据rowkey查询唯一一条记录
     * @Param: [tableName, rowKey]
     * @return: void
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
            table = conn.getTable(TableName.valueOf(tableName));
            Result r = table.get(get);
            rowMapList.add(resolveResult(r));
            logger.info("获得到rowkey: " + new String(r.getRow()));
        } catch (IOException e) {
            logger.error("Get table one data failed!");
            e.printStackTrace();
        } finally {
            try {
                if (table != null) table.close();
            } catch (IOException e) {
                logger.error("close table failed!");
                e.printStackTrace();
            }
        }
        return rowMapList;
    }

    /**
     * @Description: 单条件按查询，查询多条记录
     * @Param: [tableName]
     * @return: void
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 13:16
     */
    public static List<HashMap<String, HashMap<String, String>>> queryByCondition(String tableName, String familyName, String columnName, String columnValue) {
        ObjectArrayList<HashMap<String, HashMap<String, String>>> rowMapList = new ObjectArrayList<>(); // <familyName, <columnName, columnValue>>
        Table table = null;
        ResultScanner rs = null;
        try {
            table = conn.getTable(TableName.valueOf(tableName));
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
            try {
                if (rs != null) rs.close();
                if (table != null) table.close();
            } catch (IOException e) {
                logger.error("close table failed!");
                e.printStackTrace();
            }
        }
        return rowMapList;
    }

    /**
     * @Description: 组合条件查询
     * @Param: [tableName]
     * @return: void
     * @Author: CHEN ZUOLI
     * @Date: 2018/3/29
     * @Time: 13:26
     */
    public static List<HashMap<String, HashMap<String, String>>> queryByCondition(String tableName, String familyName, HashMap<String, String> paramMap) {
        ObjectArrayList<HashMap<String, HashMap<String, String>>> rowMapList = new ObjectArrayList<>(); // <familyName, <columnName, columnValue>>
        Table table = null;
        ResultScanner rs = null;
        try {
            table = conn.getTable(TableName.valueOf(tableName));
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
            try {
                if (rs != null) rs.close();
                if (table != null) table.close();
            } catch (IOException e) {
                logger.error("close table failed!");
                e.printStackTrace();
            }
        }
        return rowMapList;
    }

    /**
    * @Description:  解析查询hbase得到的结果，放入到HashMap中
    * @Param: [result]
    * @return: java.util.HashMap<java.lang.String,java.util.HashMap<java.lang.String,java.lang.String>>
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

}
```