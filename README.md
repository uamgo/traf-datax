# traf-datax

traf-datax 是基于***DataX*** 开发的关于esgyndb数据库的reader、writer插件，实现esgyndb与其他数据源进行互通。

为了适配不同的版本，其中有两个分支， master分支是根据 [***alibaba DataX***](https://github.com/alibaba/DataX) 代码库开发的;
hashdatax分支是根据[***HashDataInc DataX***](https://github.com/HashDataInc/DataX/) 代码库开发的;

# System Requirements

- Linux
- JDK(1.8以上，推荐1.8)  
- Python(推荐Python2.6.X)
- Apache Maven 3.x (Compile traf-datax)

# Quick Start

1、下载traf-datax源码:
``` shell
    $ git clone git@github.com:kevinxu021/traf-datax.git
```
2、通过maven打包
``` shell
    $ cd  {traf_datax_code_home}
    $ mvn -U clean package assembly:assembly -Dmaven.test.skip=true
```
打包成功，日志显示如下：
    
```
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 28.484 s
    [INFO] Finished at: 2019-09-27T16:14:52+08:00
    [INFO] ------------------------------------------------------------------------ 
```
打包成功后的DataX包位于 {traf_datax_code_home}/target/datax/datax/ ，结构如下：
``` shell
        $ cd  {traf_datax_code_home}
        $ ls  target/datax/datax/
        bin		conf		job		lib 	plugin		script		tmp
```
在{traf_datax_code_home}/target目录下还有生成一个datax.tar.gz压缩包，其中内容和上面一样。

3、补充说明
    
在{traf_datax_code_home}/target/datax/datax/plugin 的 reader 和 writer 目录下只有 esgyndbreader 和 esgyndbwriter ，
如果想要使用其它数据源，可以把相应的xxxreader、xxxwriter插件分别放到reader、writer这两个目录下。

如果之前已经安装了DataX环境，可以把 esgyndbreader 和 esgyndbwriter 放到DataX环境中的plugin相应目录下，就可以使用esgyndb了。
    
* 配置示例：
    * 第一步、创建配置文件（json格式）
    
        可以通过命令查看配置模板： python datax.py -r esgyndbreader -w esgyndbwriter
        ``` shell
        DataX (DATAX-OPENSOURCE-3.0), From Alibaba !
        Copyright (C) 2010-2017, Alibaba Group. All Rights Reserved.
        
        
        Please refer to the esgyndbreader document:
             https://github.com/alibaba/DataX/blob/master/esgyndbreader/doc/esgyndbreader.md 
        
        Please refer to the esgyndbwriter document:
             https://github.com/alibaba/DataX/blob/master/esgyndbwriter/doc/esgyndbwriter.md 
         
        Please save the following configuration as a json file and  use
             python {DATAX_HOME}/bin/datax.py {JSON_FILE_NAME}.json 
        to run the job.
        
        {
            "job": {
                "content": [
                    {
                        "reader": {
                            "name": "esgyndbreader", 
                            "parameter": {
                                "column": [], 
                                "connection": [
                                    {
                                        "jdbcUrl": [], 
                                        "table": []
                                    }
                                ], 
                                "password": "", 
                                "username": ""
                            }
                        }, 
                        "writer": {
                            "name": "esgyndbwriter", 
                            "parameter": {
                                "column": [], 
                                "connection": [
                                    {
                                        "jdbcUrl": "", 
                                        "table": []
                                    }
                                ], 
                                "password": "", 
                                "preSql": [], 
                                "session": [], 
                                "username": "", 
                                "writeMode": ""
                            }
                        }
                    }
                ], 
                "setting": {
                    "speed": {
                        "channel": ""
                    }
                }
            }
        }
        ```
        根据模板配置json如下：
        ``` json
        {
            "job": {
                "setting": {
                    "speed": {
                         "byte": 1048576
                    },
                        "errorLimit": {
                        "record": 0,
                        "percentage": 0.02
                    }
                },
                "content": [
                    {
                        "reader": {
                            "name": "esgyndbreader",
                            "parameter": {
                                "username": "admin",
                                "password": "admin",
                                "column": ["BB", "CC"],
                                "splitPk": "BB",
                                "connection": [
                                    {
                                        "table": [
                                            "trafodion.schema.AA"
                                        ],
                                        "session":[""],
                                        "jdbcUrl": [
             "jdbc:t4jdbc://localhost:23400/:"
                                        ]
                                    }
                                ]
                            }
                        },
                       "writer": {
                            "name": "esgyndbwriter",
                            "parameter": {
                                "username": "admin",
                                "password": "admin",
                                "column": ["BB", "CC"],
                                "preSql": ["delete from trafodion.schema.DD where BB = '5'"],
                                "postSql": ["delete from trafodion.schema.DD where BB = '4'"],
                                "connection": [
                                    {
                                        "table": [
                                            "trafodion.schema.DD"
                                        ],
                                        "session":[""],
                                        "jdbcUrl": "jdbc:t4jdbc://localhost:23400/:", 
                                    }
                                ]
                            }
                        }
                    }
                ]
            }
        }
        ```
        * 第二步：启动DataX
            
            ``` shell
            $ cd {YOUR_DATAX_DIR_BIN}
            $ python datax.py esgyndb2esgyndb.json
            ```
            
            同步结束，显示日志如下：
            
            ``` shell
            ...
            2019-09-27 17:18:38.851 [job-0] INFO  JobContainer - 
            任务启动时刻                    : 2019-09-27 17:18:23
            任务结束时刻                    : 2019-09-27 17:18:38
            任务总计耗时                    :                 15s
            任务平均流量                    :                2B/s
            记录写入速度                    :              0rec/s
            读出记录总数                    :                   5
            读写失败总数                    :                   0
            ```
        
# FAQ

Q: 当前插件支持哪个版本的esgyndb

A: 当前插件中使用的是jdbcT4-2.7.0.jar，如果想要替换其它版本的

    (1) 在esgyndbreader、 esgyndbwriter项目的pom文件中修改jdbc依赖

    (2) 替换相应lib目录下的 jdbc jar包
    
    (3) 重新编译打包
    
