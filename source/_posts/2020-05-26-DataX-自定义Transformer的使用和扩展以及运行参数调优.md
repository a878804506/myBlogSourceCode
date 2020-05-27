---
title: DataX-自定义Transformer的使用和扩展以及运行参数调优
top: true
cover: true
date: 2020-05-26 18:13:12
categories: DataX
tags: 
  - java
  - 技术
---

> **<font color=green>写在前面</font>**：在使用数据同步工具的时候，在业务上往往不会是简单的从A库到B库，我公司现在使用的数据交换平台也有函数转换功能，最近玩DataX的时候发现她自带了字段转换预处理功能，特来实战一下。

### 一、本篇教程侧重点导读
> 1. 使用Transformer时的job.json的配置示例；
> 2. dx_substr的使用示例；
> 3. dx_pad的使用示例；
> 4. dx_replace的使用示例；
> 5. dx_filter的使用示例；
> 6. dx_groovy执行自定义代码；
> 7. 扩展Transformer；
> 8. DataX的同步性能调优；

### 二、本篇教程用的软件、技术和说明
> 1. jdk版本：1.8.0_202-b08；
> 2. Python版本：2.7.18（官方推荐2.6.X）；
> 3. Maven版本；3.6.0 ；
> 4. DataX是直接拉取的master分支上的源码；


### 三、使用Transformer时的job.json的配置示例
1. git上面DataX也对Transformer做了使用介绍，地址是：[**<font color=green>跳转</font>**](https://github.com/alibaba/DataX/blob/master/transformer/doc/transformer.md "跳转")
2. 使用转换配置的完整示例：该示例是字符串的截取示例，后面会对每一种的字段处理做详细介绍和demo
````json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 2
      },
      "errorLimit": {
        "record": 10000,
        "percentage": 1
      }
    },
    "content": [
      {
		// 字段转换部分
		"transformer": [
          {
		    // 使用字段截取转换
            "name": "dx_substr",
            "parameter": {
			  // 操作读取出来的record的第一列
              "columnIndex": 0,
			  // 意思是截取第0到4个字符
              "paras": ["0","4"]
            }
          }
        ],
        "reader": {
          "name": "mysqlreader",
          "parameter": {
            "username": "root",
            "password": "root",
            "column": [
              "MP_ID",
              "LOAD_TIME",
              "DATA_TIME",
              "POS_P_E_TOTAL",
              "REV_P_E_TOTAL",
              "GROUP_P_E_TOTAL",
              "GROUP_Q_E_1",
              "GROUP_Q_E_2",
              "QUAD_1_Q_E_TOTAL",
              "QUAD_2_Q_E_TOTAL",
              "QUAD_3_Q_E_TOTAL",
              "QUAD_4_Q_E_TOTAL",
              "DATA_FLAG"
            ],
            "splitPk": "MP_ID",
            "connection": [
              {
                "table": [
                  "dr_e_raw_hour_202004_debug"
                ],
                "jdbcUrl": [
                  "jdbc:mysql://192.168.1.202:3306/test"
                ]
              }
            ]
          }
        },
        "writer": {
          "name": "mysqlwriter",
          "parameter": {
            "username": "root",
            "password": "root",
            "column": [
              "MP_ID",
              "LOAD_TIME",
              "DATA_TIME",
              "POS_P_E_TOTAL",
              "REV_P_E_TOTAL",
              "GROUP_P_E_TOTAL",
              "GROUP_Q_E_1",
              "GROUP_Q_E_2",
              "QUAD_1_Q_E_TOTAL",
              "QUAD_2_Q_E_TOTAL",
              "QUAD_3_Q_E_TOTAL",
              "QUAD_4_Q_E_TOTAL",
              "DATA_FLAG"
            ],
            "connection": [
              {
                "table": [
                  "dr_e_raw_hour_202004_debug"
                ],
                "jdbcUrl": "jdbc:mysql://192.168.1.202:3306/test1"
              }
            ]
          }
        }
      }
    ]
  }
}
````

### 四、dx_substr的使用示例
1. 配置demo
````json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 2
      },
      "errorLimit": {
        "record": 10000,
        "percentage": 1
      }
    },
    "content": [
      {
		// 字段转换部分
		"transformer": [
          {
		    // 使用字段截取转换
            "name": "dx_substr",
            "parameter": {
			  // 操作读取出来的record的第一列
              "columnIndex": 0,
			  // 意思是截取第0到4个字符
              "paras": ["0","4"]
            }
          }
        ],
        "reader": {
          // 略去 reader
        },
        "writer": {
          // 略去 writer
        }
      }
    ]
  }
}
````
2. 运行后的效果
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200526/4.2.png"  align=left/>

### 五、dx_pad的使用示例
1. 配置demo
````json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 2
      },
      "errorLimit": {
        "record": 10000,
        "percentage": 1
      }
    },
    "content": [
      {
		// 字段转换部分
		"transformer": [
          {
            "name": "dx_pad",
            "parameter": {
              "columnIndex": 0,
			  // 第一个参数是在字段左边补位还是右边补位，可选参数有 l:左边 、r:右边
			  // 第二个参数是目标字段的长度
			  // 第三个参数是需要补位的字符
			  // 返回：如果源字符串长度小于目标字段长度，按照位置添加pad字符后返回。如果长于，直接截断（都截右边）。如果字段为空值，转换为空字符串进行pad，即最后的字符串全是需要pad的字符
              "paras": ["l","14","x"]
            }
          }
        ],
        "reader": {
          // 略去 reader
        },
        "writer": {
          // 略去 writer
        }
      }
    ]
  }
}
````
2. 运行后的效果
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200526/5.2.png"  align=left/>


### 六、dx_replace的使用示例
1. 配置demo
````json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 2
      },
      "errorLimit": {
        "record": 10000,
        "percentage": 1
      }
    },
    "content": [
      {
		// 字段转换部分
		"transformer": [
          {
            "name": "dx_replace",
            "parameter": {
              "columnIndex": 0,
              //第一个参数：字段值的开始位置。
              //第二个参数：需要替换的字段长度。
              //第三个参数：需要替换的字符串。
              //返回： 从字符串的指定位置（包含）替换指定长度的字符串。如果开始位置非法抛出异常。如果字段为空值，直接返回（即不参与本transformer）
              "paras": ["3","7","x"]
            }
          }
        ],
        "reader": {
          // 略去 reader
        },
        "writer": {
          // 略去 writer
        }
      }
    ]
  }
}
````
2. 运行后的效果
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200526/6.2.png"  align=left/>

### 七、dx_filter的使用示例
1. 配置demo
````json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 2
      },
      "errorLimit": {
        "record": 10000,
        "percentage": 1
      }
    },
    "content": [
      {
		// 字段转换部分
		"transformer": [
          {
            "name": "dx_filter",
            "parameter": {
              "columnIndex": 0,
              //第一个参数：运算符，支持一下运算符：like, not like, >, =, <, >=, !=, <=
              //第二个参数：正则表达式（java正则表达式）、值。
              //返回：
              //1、如果匹配正则表达式，返回Null，表示过滤该行。不匹配表达式时，表示保留该行。（注意是该行）。对于>=<都是对字段直接compare的结果.
              //2、like ， not like是将字段转换成String，然后和目标正则表达式进行全匹配。
              //, =, <, >=, !=, <= 对于DoubleColumn比较double值，对于LongColumn和DateColumn比较long值，其他StringColumn，BooleanColumn以及ByteColumn均比较的是StringColumn值。
              //3、如果目标colunn为空（null），对于如果目标colunn为空（null），对于 = null的过滤条件，将满足条件，被过滤。！=null的过滤条件，null不满足过滤条件，不被过滤。 like，字段为null不满足条件，不被过滤，和not like，字段为null满足条件，被过滤。
              "paras": ["like","102000000008"]
            }
          }
        ],
        "reader": {
          // 略去 reader
        },
        "writer": {
          // 略去 writer
        }
      }
    ]
  }
}
````
2. 运行后的效果
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200526/7.2.png"  align=left/>


### 八、dx_groovy执行自定义代码
1. dx_groovy就比较有意思了，它可以执行自定义的代码，使用这个转换可以实现上述4种类型的转换，你可以获取到读取出来的每一条记录（record），也就可以得到每条记录的每一列，然后做你想做的事情。
2. 如下json配置，业务上需要取LOAD_TIME和DATA_TIME时间字段做比较，如果LOAD_TIME比DATA_TIME还要晚一些，说明这条数据就要废弃掉，就把DATA_FLAG标识为0。
````json
{
  "job": {
    "setting": {
      "speed": {
        "channel": 2
      },
      "errorLimit": {
        "record": 10000,
        "percentage": 1
      }
    },
    "content": [
      {
		// 字段转换部分
		"transformer": [
          {
		    //自定义转换
            "name": "dx_groovy",
            "parameter": {
              "code": "Column loadTimeColumn = record.getColumn(1);
			  Column dataTimeColumn = record.getColumn(2);
			  Date loadTime = loadTimeColumn.asDate();
			  Date dataTime = dataTimeColumn.asDate();
			  if(loadTime.after(dataTime)){
			  record.setColumn(12,new LongColumn(0l));
			  return record;
			  }else{return record;}",
              "extraPackage": [
                ""
              ]
            }
          }
        ],
        "reader": {
          // 略去 reader
        },
        "writer": {
          // 略去 writer
        }
      }
    ]
  }
}
````

3. 运行后的效果
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200526/8.3.png"  align=left/>

4. extraPackage参数是需要导入的包名，阅读`GroovyTransformer`类源码可知道，在使用dx_groovy的时候会自动加载如下常用类：
 <img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200526/8.4.png"  align=left/>
 所以说在使用dx_groovy时，需要引用到上述包的时候，可以不用写包的引用，特别是util包；
 
5. 值得注意的是：dx_groovy它引用了这个包`import static com.alibaba.datax.core.transport.transformer.GroovyTransformerStaticUtil.*`,而这个静态类中目前是空的，阿里的大佬已经为你考虑到：当你需要编写大量逻辑的时候，你可以直接写在这个类中（当然要声明成静态方法），然后编译DataX，当再次使用dx_groovy时，可直接`GroovyTransformerStaticUtil.方法`的的形式调用，来执行你的逻辑！

### 九、扩展Transformer
 如果使用dx_groovy还不够酷炫，还不能满足业务需求，还可以对Transformer模块进行扩展，编写自己的特有的Transformer模块，方法如下：
 1. 下载DataX源码，在com.alibaba.datax.core.transport.transformer包下，新建一个类继承Transformer，可以参照已有的类（FilterTransformer、GroovyTransformer等等）
 2. 在TransformerRegistry类的静态代码块中注册你的类
 3. 编译打包DataX，在job.json中使用`"transformer": []`模块

### 十、DataX的同步性能调优
1. 全局参数
````json
{
   "core":{
        "transport":{
            "channel":{
                "speed":{
                    "channel": 2, ## 数据导入的并发度
                    "record":-1, ##解除对读取行数的限制
                    "byte":-1, ##解除对字节的限制
                    "batchSize":2048 ##每次读取batch的大小
                }
            }
        }
    },
    "job":{
            ...
        }
    }
````
2. 局部参数
````json
{
    "job": {
        ....
        },
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "username": "root",
                        "password": "root",
                        "column": [*],
                        "splitPk": "db_id", ## 必须指定切分键并发才生效
                        "connection": [
                            ......
                                ]
                            }
                        ]
                    }
                },
               "writer": {
````
 1. channel增大，可根据服务器的实际配置修改datax.py，提升-Xms与-Xmx，来防止OOM。
 2. DEFAULT_JVM = "-Xms1g -Xmx1g -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=%s/log" % (DATAX_HOME)
 3. channel并不是越大越好，过分大反而会影响宿主机的性能。

3. jvm调优，可根据服务器条件为job设置合适等jvm大小
 `$ python ./bin/datax.py  --jvm="-Xms512m -Xmx512m" ./job/stream2stream.json`
 `$ python ./bin/datax.py  --jvm="-Xms1G -Xmx1G" ./job/job.json`
