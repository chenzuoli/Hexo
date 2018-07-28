---
title: 操作mysql工具类
date: 2018-04-02 20:02:20
tags: [mysql,utils]
categories: 工具类
---
下面介绍的是操作mysql的工具类，集成增删改查等功能方法，使用dbcp数据库连接池，让你的程序更高效。具体请看详情。
<!-- more -->
备注：代码环境jdk8（jdk7也可以）
# maven项目依赖
```
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.41</version>
</dependency>
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-dbcp2</artifactId>
	<version>2.1.1</version>
</dependency>
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-pool2</artifactId>
	<version>2.4.2</version>
</dependency>
```
# 数据库连接配置文件
配置文件jdbc.properties放置在项目resources目录下，配置如下：
```
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/zs1?useSSL=false
username=root
password=root
initialSize=10
maxIdle=5
minIdle=2
autoReconnect=true
autoReconnectForPools=true
```
# 具体代码
```
import com.payegis.czl.model.QueryLogHistory;
import org.apache.commons.dbcp2.BasicDataSourceFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.sql.DataSource;
import java.io.IOException;
import java.sql.*;
import java.util.*;

/**
 * User: 陈作立
 * Date: 2018/2/2
 * Time: 13:39
 * Description: 操作mysql数据库工具类
 * Ps: mysql
 */
public class DBCPUtil {
    private static Logger loger = LoggerFactory.getLogger(DBCPUtil.class);
    private static DataSource dataSource = null;

    static {
        loger.info("---------开始初始化数据库连接池---------");
        Properties prop = new Properties();
        try {
            prop.load(DBCPUtil.class.getClassLoader().getResourceAsStream("jdbc.properties"));
            dataSource = BasicDataSourceFactory.createDataSource(prop);
        } catch (IOException e) {
            loger.error("---------加载[jdbc.properties]失败---------", e);
        } catch (Exception e) {
            loger.error("----------初始化数据库连接池异常失败---------", e);
        }
        loger.info("---------数据库连接池初始化完成---------");
    }

    /**
     * 获取数据库连接
     *
     * @return
     */
    public static Connection getConnection() {
        Connection conn = null;
        if (conn != null) {
            return conn;
        }
        try {
            conn = dataSource.getConnection();
        } catch (SQLException e) {
            loger.error("---------数据库连接池获取连接异常---------", e);
        }
        return conn;
    }

    /**
     * 关闭数据库连接
     *
     * @param connection 数据库连接
     */
    public static void close(Connection connection) {
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                loger.error("---------关闭Connection异常---------", e);
            }
        }
    }

    /**
     * 关闭数据库连接
     *
     * @param conn 数据库连接
     * @param stat 预编译
     */
    public static void close(Connection conn, Statement stat) {
        try {
            if (stat != null) {
                stat.close();
            }
        } catch (SQLException e) {
            loger.error("---------关闭Connection、PreparedStatement异常---------", e);
        } finally {
            close(conn);
        }
    }

    /**
     * 关闭数据库连接
     *
     * @param conn 数据库连接
     * @param stat 预编译
     * @param rs   结果集
     */
    public static void close(Connection conn, Statement stat, ResultSet rs) {
        try {
            if (rs != null) {
                rs.close();
            }
        } catch (SQLException e) {
            loger.error("---------关闭ResultSet异常---------", e);
        } finally {
            close(conn, stat);
        }
    }

    /**
     * 执行查询
     *
     * @param sql
     * @param params
     * @return
     */
    public static List<Map<String, Object>> executeQuery(String sql, Object... params) {
        List<Map<String, Object>> rowDataList = new ArrayList<Map<String, Object>>();
        Connection conn = null;
        PreparedStatement stat = null;
        ResultSet resultSet = null;
        try {
            conn = getConnection();
            stat = conn.prepareStatement(sql);
            stat.setFetchSize(10000);
            setStatParams(stat, params);
            resultSet = stat.executeQuery();
            rowDataList = getResultList(resultSet);
        } catch (SQLException e) {
            loger.error("---------数据查询异常[" + sql + "]---------", e);
        } finally {
            close(conn, stat, resultSet);
        }
        return rowDataList;
    }

    /**
     * 更新数据
     *
     * @param sql    sql语句
     * @param params 参数
     * @return 更新成功:true 更新失败:false
     */
    public static boolean executeUpdate(String sql, Object... params) {
        boolean isUpdated = false;
        Connection conn = null;
        PreparedStatement stat = null;
        try {
            conn = getConnection();
            conn.setAutoCommit(false);
            stat = conn.prepareStatement(sql);
            setStatParams(stat, params);
            int updatedNum = stat.executeUpdate();
            isUpdated = updatedNum == 1;
            conn.commit();
        } catch (SQLException e) {
            try {
                conn.rollback();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
            loger.error("---------更新失败! sql:[" + sql + "], params:[" + Arrays.toString(params) + "]---------", e);
        } finally {
            close(conn, stat);
        }
        return isUpdated;
    }

    /**
     * 执行批处理
     *
     * @param sqlList sql语句集合
     * @return
     */
    public static boolean executeBatch(List<String> sqlList) {
        if (sqlList == null || sqlList.isEmpty()) {
            return true;
        }
        Connection conn = null;
        Statement stat = null;
        try {
            conn = getConnection();
            conn.setAutoCommit(false);
            stat = conn.createStatement();
            for (String sql : sqlList) {
                stat.addBatch(sql);
            }
            stat.executeBatch();
            conn.commit();
            return true;
        } catch (SQLException e) {
            try {
                conn.rollback();
                loger.error("---------批处理异常，执行回滚---------");
            } catch (SQLException e1) {
                loger.error("---------回滚异常---------", e1);
            }
            loger.error("---------执行批处理异常---------");
            loger.error("---------批处理异常sql：" + Arrays.toString(sqlList.toArray()));
        } finally {
            try {
                if (conn != null) {
                    conn.setAutoCommit(true);
                }
            } catch (SQLException e) {
                loger.error("---------设置自动提交异常---------", e);
            }
            close(conn, stat);
        }
        return false;
    }

    /**
     * 获取列名及数据
     *
     * @param rs 数据集
     * @return
     */
    private static List<Map<String, Object>> getResultList(ResultSet rs) throws SQLException {
        List<Map<String, Object>> rowDataList = new ArrayList<Map<String, Object>>();
        List<String> colNameList = getColumnName(rs);
        while (rs.next()) {
            Map<String, Object> rowData = new HashMap<String, Object>();
            for (String colName : colNameList) {
                rowData.put(colName, rs.getObject(colName));
            }
            if (!rowData.isEmpty()) {
                rowDataList.add(rowData);
            }
        }
        return rowDataList;
    }

    /**
     * 获取列名
     *
     * @param rs 数据集
     * @return
     */
    private static List<String> getColumnName(ResultSet rs) throws SQLException {
        List<String> columnList = new ArrayList<String>();
        try {
            ResultSetMetaData metaData = rs.getMetaData();
            int columnCount = metaData.getColumnCount();
            for (int i = 1; i <= columnCount; i++) {
                columnList.add(metaData.getColumnName(i));
            }
        } catch (SQLException e) {
            loger.info("------获取表列表异常------", e);
            throw e;
        }
        return columnList;
    }

    /**
     * 设置参数
     *
     * @param stat   预编译
     * @param params 参数
     */
    private static void setStatParams(PreparedStatement stat, Object... params) throws SQLException {
        if (stat != null && params != null) {
            try {
                for (int len = params.length, i = 1; i <= len; i++) {
                    stat.setObject(i, params[i - 1]);
                }
            } catch (SQLException e) {
                loger.error("------设置sql参数异常---------");
                throw e;
            }
        }
    }

}
```
好了，到这里就结束了，这个类基本可以满足操作mysql的需求了，大家放心使用吧，如果有什么问题，或者可以优化的地方，欢迎大家email我chenzuoli709@gmail.com