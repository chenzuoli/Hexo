---
title: airbnb开源调度系统airflow的一些命令及使用方法
date: 2019-08-02 00:06:33
tags: airflow
categories: 调度系统
notebook: 笔记
---

python写的调度系统，用python脚本，动态生成dag，跨dag依赖，是一个不错的调度系统，下面介绍一些我使用过程中用到的命令和问题的解决方案。

<!-- more -->

# 1.operator
```
    BashOperator
    PythonOperator
    EmailOperator
    HTTPOperator
    SqlOperator
    Sensor
    DockerOperator
    HiveOperator
```
# 2.给DAG实例传递参数
执行命令
```
airflow trigger_dag example_passing_params_via_test_command -c '{"foo":"bar"}'
```
代码获取变量：
```
def my_py_command(ds, **kwargs):
logging.info(kwargs)
logging.info(kwargs.get('dag_run').conf.get('foo'));
```
# 3.填补数据
```
#清除dag在这段时间内的状态，清除后airflow会自动启动这些任务，如果dag设置了catchup=True;dependency_on_past=True;那么dag会按照时间顺序一天一天跑任务，这对于修补数据很有用哦
airflow clear db2src_usersdb_byshell -s 2018-12-01 -e 2018-12-04
#回填数据，当新建一个dag，需要补跑以前的数据，回填命令是个不错的选择
airflow backfill db2src_usersdb_byshell -s 2018-12-03 -e 2018-12-04
```
# 4.根据depend_on_past
True or False来判断是否需要依赖start_time前段时间跑的相同的任务情况来运行现在的任务。
# 5.airflow卡住的问题
连接元数据mysql库：select * from task_instance where state = 'running';
# 6.airflow自带变量：
```
| Variable                           | Description |
| :------:                           | :---------: |
|{{ ds }}                            |the execution date as YYYY-MM-DD|
|{{ ds_nodash }}                     |the execution date as YYYYMMDD|
|{{ yesterday_ds }}                  |yesterday’s date as YYYY-MM-DD|
|{{ yesterday_ds_nodash }}           |yesterday’s date as YYYYMMDD|
|{{ tomorrow_ds }}                   |tomorrow’s date as YYYY-MM-DD|
|{{ tomorrow_ds_nodash }}            |tomorrow’s date as YYYYMMDD|
|{{ ts }}                            |same as execution_date.isoformat()|
|{{ ts_nodash }}                     |same as ts without - and :|
|{{ execution_date }}                |the execution_date, (datetime.datetime)|
|{{ prev_execution_date }}           |the previous execution date (if available) (datetime.datetime)|
|{{ next_execution_date }}           |the next execution date (datetime.datetime)|
|{{ dag }}                           |the DAG object|
|{{ task }}                          |the Task object|
|{{ macros }}                        |a reference to the macros package, described below|
|{{ task_instance }}                 |the task_instance object|
|{{ end_date }}                      |same as {{ ds }}|
|{{ latest_date }}                   |same as {{ ds }}|
|{{ ti }}                            |same as {{ task_instance }}|
|{{ params }}                        |a reference to the user-defined params dictionary|
|{{ var.value.my_var }}              |global defined variables represented as a dictionary|
|{{ var.json.my_var.path }}          |global defined variables represented as a dictionary with deserialized JSON object, append the path to the key within the JSON object|
|{{ task_instance_key_str }}         |a unique, human-readable key to the task instance formatted {dag_id}_{task_id}_{ds}              |
|conf                                |the full configuration object located at airflow.configuration.conf which represents the content of your airflow.cfg|
|run_id                              |the run_id of the current DAG run|
|dag_run                             | a reference to the DagRun object|
|test_mode                           | whether the task instance was called using the CLI’s test subcommand|
```

# 7.导入导出airflow变量
```
    airflow variables --import variable.json
    airflow variables --export variable.txt
```
# 8.Template Not Found
TemplateNotFound: sh /data/airflow_dag/dags_migration/sh/export-variables.sh
这是由于airflow使用了jinja2作为模板引擎导致的一个陷阱，当使用bash命令的时候，尾部必须加一个空格
```
t2 = BashOperator(
task_id=‘sleep‘,
bash_command="/home/batcher/test.sh", // This fails with `Jinja template not found` error
#bash_command="/home/batcher/test.sh ", // This works (has a space after)
dag=dag)
```
# 9. 手动触发dag运行
```
airflow trigger_dag dag_id -r RUN_ID -e EXEC_DATE
```
# 10. 手动触发task运行
```
airflow run dag_id task_id EXEC_DATE
```

# 11. "Failed to fetch log file from worker"
查看task_instance中hostname字段，存储的均为localhost；
分析：修改/etc/hosts文件，删除127.0.0.1  hostname映射；worker log服务获取到hostname后，映射到ip后得到127.0.0.1，故无法访问到log。

# 12. airflow中每个task对应的执行priority计算方式
dummy2 = DummyOperator(
    task_id='dummy_' + src_db,
    pool='db',
    priority_weight=weight,
    dag=dag
)
所有后置依赖的priority_weight之和，最后一个任务的priority_weight如果没有自定义，默认为1，这样，在同一个pool中做到了任务优先运行；
