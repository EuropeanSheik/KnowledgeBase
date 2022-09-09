# 一、LogStash的安装

### 1. Java的安装

1. 卸载虚拟机自带的Java

   ```bash
   rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps
   ```

2. 上传java安装包到`/export/server/`目录下

3. 解压java压缩包

   ```bash
   tar -zxvf /export/server/jdk-17_linux-x64_bin.tar.gz
   ```

4. 删除安装包

   ```bash
   rm -rf /export/server/jdk-17_linux-x64_bin.tar.gz
   ```

5. 重命名文件夹

   ```bash
   mv jdk-17.0.2/ jdk
   ```

6. 配置环境变量

   ```bash
   vim /etc/profile
   ```

   ```bash
   # Java_Path
   export JAVA_HOME=/export/server/jdk
   export PATH=$PATH:$JAVA_HOME/bin
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   # LogStash_Java_Path
   export LS_JAVA_HOME=/export/server/jdk
   ```

7. 重载环境变量

   ```bash
   source /etc/profile
   ```

8. 验证是否安装成功

   ```bash
   java -version
   ```

9. 安装成功
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/329842fcad144bb4a22f5eb0026785ce.png)

### 2. 安装LogStash

1. 下载LogStash安装包
   [LogStash官网下载](https://www.elastic.co/cn/downloads/logstash)

2. 将安装包上传至`/export/server/`目录下

3. 解压压缩包

   ```bash
   tar -xvf /export/server/logstash-8.1.2-linux-x86_64.tar.gz
   ```

4. 删除压缩包

   ```bash
   rm -rf /export/server/logstash-8.1.2-linux-x86_64.tar.gz
   ```

5. 重命名文件夹

   ```bash
   mv /export/server/logstash-8.1.2/ /export/server/logstash
   ```

6. 运行示例检验是否安装成功

   ```bash
   /export/server/logstash/bin/logstash -e 'input { stdin {} } output { stdout {} }'
   ```

7. 安装成功
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/9686d1fe72714d70bc69c901303a847b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARXVyb3BlYW5TaGVpaw==,size_20,color_FFFFFF,t_70,g_se,x_16)

# 二、input插件

使用输入将数据输入LogStash

### 1.标准输入（控制台输入）

```properties
input {
    stdin {
    }
}
```

### 2.syslog

```properties
input {
    syslog {
        port => "514"
    }
}

```

### 3.twitter

```properties
input {
	twitter { 
        consumer_key => "enter_your_consumer_key_here" 
        consumer_secret => "enter_your_secret_here"
        关键字=> [ "cloud" ] 
        oauth_token => "enter_your_access_token_here" 
        oauth_token_secret => "enter_your_access_token_secret_here" 
	}
}
```

### 4.beats

```properties
input {
     beats {
        port => "5044"
    }
}
```

# 三、filter插件

filter是LogStash管道中的中间处理设备。如果事件符合特定条件，可以将过滤器与条件结合起来对事件执行操作

### 1. Grok：能够将非结构化日志数据解析为结构化和可查询的内容

[官方教程](https://www.elastic.co/guide/en/logstash/8.1/plugins-filters-grok.html)

```properties
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
```

### 2.Geoip：查找 IP 地址，从地址中获取地理位置信息，并将该位置信息添加到日志中。

[官方教程](https://www.elastic.co/guide/en/logstash/8.1/plugins-filters-geoip.html)

```properties
filter {
    geoip {
    	source => "clientip"
        }
}
```

# 四、output插件

output是LogStash的最后阶段。一个事件可以通过多个输出，但是一旦所有输出处理完成，事件就完成了它的执行。

### 1.标准输出

```bash
output {
    stdout {
        codec => rubydebug
    }
}
```

### 2.ElasticSearch

```bash
output {
    elasticsearch {
        hosts => [ "host:9200" ]
    }
}
```

### 3.文件

```bash
input {
    file {
        path => "/path/to/target/file"
        }
}
```

### 4.kafka

```bash
output{
    kafka{
        # kafka topic的名称
        topic_id => "kafka-topic"
    }
}
```

# 五、codec

codec是基础的流过滤器，可以作为输入或输出的一部分进行操作。编码器能够将消息的传输与序列化过程分开

### 1. json：以JSON格式编码或解码数据

### 2. multiline：将多行文本事件合并为单个事件

# 六、配置行可选参数

### 1. `--config.test_and_exit`

```bash
# 解析配置文件并报告错误
logstash -f test.conf --config.test_and_exit
```

### 2. `--config.reload.automatic`

```bash
# 启用自动重载配置功能
logstash -f test.conf --config.reload.automatic
```

# 六、生产经验

### 1. 多个输入过滤输出

```properties
input {
	beats {
		port => "5044"
	}
	syslog {
		port => "514"
	}
}
filter {
	grok {
		match => {
			"message" => "%{DATA:head} - - - %{DATA:new_message}\n"
		}
	}
	mutate {
		split => {
			"head" => " "
		}
	}
}
output {
	stdout {}
	elasticsearch {
		hosts => ["http://192.168.10.101:9200"]
		index => "jg_zjsj_new_202207"
	}
}
```

