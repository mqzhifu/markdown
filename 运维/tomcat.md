# tomcat

cd /install

wget https://downloads.apache.org/tomcat/tomcat\-8/v8.5.81/bin/apache\-tomcat\-8.5.81.zip

unzip apache\-tomcat\-8.5.81.zip

mv apache\-tomcat\-8.5.81 /soft/tomcat

cd /soft/tomcat

bin/startup.sh

bin/shutdown.sh

vim conf/tomcat\-users.xml

```
role rolename="manager-gui"/>
<user password="ckckarar" roles="manager-gui" username="ckadmin"/>
```

vim conf/server.xml

```
 <Host name="localhost"  appBase="/data/www/tomcat_webapps" unpackWARs="true" autoDeploy="true">
```

vim tomcat\_webapps/manager/META\-INF/context.xml

```
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
          allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|\d+\.\d+\.\d+\.\d+" />
```

wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx\_prometheus\_javaagent/0.17.0/jmx\_prometheus\_javaagent\-0.17.0.jar

wget https://github.com/prometheus/jmx\_exporter/blob/master/example\_configs/tomcat.yml

cp jmx\_prometheus\_javaagent\-0.17.0.jar tomcat.yml /soft/tomcat/bin

vim bin/catalina.sh

```
JAVA_OPTS="-javaagent:/soft/tomcat/bin/jmx_prometheus_javaagent-0.17.0.jar=30018:/soft/tomcat/bin/tomcat.yaml"
```

添加prometheus 配置

添加grafana图形
