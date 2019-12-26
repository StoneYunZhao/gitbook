# Local Debug

### 环境

* 系统操作：macOS Mojave
* JDK：1.12，`export JAVA_HOME=$(/usr/libexec/java_home -v 1.12)`
* Elasticsearch版本：7.3.1

### clone 源码

```bash
cd ~/es_data
git clone https://github.com/elastic/elasticsearch.git
cd elasticsearch
git checkout -b test v7.3.1
```

### 导入 idea 环境

```bash
./gradlew idea
```

Import Project =&gt; 选择 elasticsearch 目录 =&gt; Import project from external model \(Gradle\) =&gt; Use auto-import

![](../../.gitbook/assets/image%20%2816%29.png)

### 启动方式一

```bash
# 这一步网络下载可能很慢，必要时使用代理  
# export http_proxy=http://127.0.0.1:1087
# export https_proxy=http://127.0.0.1:1087;
./gradlew run --debug-jvm
```

等运行到如下界面，就可以用 idea 执行 Run =&gt; Attach to process：

![](../../.gitbook/assets/image%20%28104%29.png)

缺点：

* 启动很慢，第一次需要下载 jdk 等依赖文件；
* 日志窗口在 terminal，不在 debug 的 console 窗口；

优点：操作简单

### 启动方式二

Elasticsearch 启动类：server/src/main/java/org/elasticsearch/bootstrap/Elasticsearch，继承`EnvironmentAwareCommand`。

```java
public abstract class EnvironmentAwareCommand extends Command {
    @Override
    protected void execute(Terminal terminal, OptionSet options) throws Exception {
        putSystemPropertyIfSettingIsMissing(settings, "path.data", "es.path.data");
        putSystemPropertyIfSettingIsMissing(settings, "path.home", "es.path.home");
        putSystemPropertyIfSettingIsMissing(settings, "path.logs", "es.path.logs");
    }
    
    protected Environment createEnv(final Map<String, String> settings) throws UserException {
        final String esPathConf = System.getProperty("es.path.conf");
    }
}
```

可以看出我们需要配置以上四个配置：

```bash
cd ~/es_data
mkdir config data logs home

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.3.1-darwin-x86_64.tar.gz
tar zxvf elasticsearch-7.3.1-darwin-x86_64.tar.gz
cp -R elasticsearch-7.3.1/modules home/
cp -R elasticsearch-7.3.1/plugins home/

cp -R elasticsearch/distribution/src/config/* config/

vi config/elasticsearch.yml
cluster.name: my-application
node.name: node-1
#${path.data}
#${path.logs}

vi config/elasticsearch.policy
grant {
  permission java.lang.RuntimePermission "createClassLoader";
};
```

然后在 idea 做如下配置后，运行 Elasticsearch 的 main 方法即可：

![](../../.gitbook/assets/image%20%28139%29.png)

```text
-Des.path.data=~/es_idea/data
-Des.path.home=~/es_idea/home
-Des.path.logs=~/es_idea/logs
-Des.path.conf=~/es_idea/config
-Djava.security.policy=~/es_idea/config/elasticsearch.policy
-Dlog4j2.disable.jmx=true
```

