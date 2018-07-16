# AMQ

## Doc

* [Apache.ActiveMQ](http://activemq.apache.org/)
* https://access.redhat.com/documentation/en-us/red_hat_amq/6.3/
* [installation](https://access.redhat.com/documentation/en-us/red_hat_jboss_a-mq/6.3/html/installation_guide/installingzip)

## Apache ActiveMQ

[Installation](http://activemq.apache.org/getting-started.html)

```sh
root@ip-172-31-11-25: ~/amq/apache-activemq-5.15.4/bin # ./activemq start
INFO: Loading '/root/amq/apache-activemq-5.15.4//bin/env'
INFO: Using java '/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.172-1.b11.el7.x86_64/bin/java'
INFO: Starting - inspect logfiles specified in logging.properties and log4j.properties to get details
INFO: pidfile created : '/root/amq/apache-activemq-5.15.4//data/activemq.pid' (pid '3258')

# ss -tnlp | grep 3258
LISTEN     0      50          :::8161                    :::*                   users:(("java",pid=3258,fd=142))
LISTEN     0      50          :::45189                   :::*                   users:(("java",pid=3258,fd=13))
LISTEN     0      128         :::5672                    :::*                   users:(("java",pid=3258,fd=130))
LISTEN     0      128         :::61613                   :::*                   users:(("java",pid=3258,fd=131))
LISTEN     0      50          :::61614                   :::*                   users:(("java",pid=3258,fd=133))
LISTEN     0      128         :::61616                   :::*                   users:(("java",pid=3258,fd=129))
LISTEN     0      128         :::1883                    :::*                   users:(("java",pid=3258,fd=132))

```

Open port 8161 and port 61613 for inbound traffic for the security group used for the ec2 instance.

[Client library](http://activemq.apache.org/cross-language-clients.html): [go-stomp](https://github.com/go-stomp/stomp)

[my-code-example](https://github.com/hongkailiu/test-go/blob/master/stomp/main.go)

```sh
$ go build -o build/stomp ./stomp/
$ ./build/stomp -server ec2-54-201-211-200.us-west-2.compute.amazonaws.com:61613
```

Notice those:

* persistent msg: [`persistent:true`](https://activemq.apache.org/stomp.html)
* data store: Search for `persistenceAdapter` section in `conf/activemq.xml`.

## JBoss AMQ

```sh
# unzip jboss-a-mq-6.3.0.redhat-187.zip 
# cd jboss-a-mq-6.3.0.redhat-187/
# vi etc/users.properties 
# ./bin/start
### get jboss pid
# jps -lv | grep karaf

# ss -tnlp | grep 3385
LISTEN     0      1         ::ffff:127.0.0.1:39838                   :::*                   users:(("java",pid=3385,fd=233))
LISTEN     0      50          :::8101                    :::*                   users:(("java",pid=3385,fd=253))
LISTEN     0      128         :::5672                    :::*                   users:(("java",pid=3385,fd=311))
LISTEN     0      50          :::1099                    :::*                   users:(("java",pid=3385,fd=244))
LISTEN     0      50          :::35437                   :::*                   users:(("java",pid=3385,fd=26))
LISTEN     0      50          :::61614                   :::*                   users:(("java",pid=3385,fd=315))
LISTEN     0      128         :::61616                   :::*                   users:(("java",pid=3385,fd=309))
LISTEN     0      50          :::8181                    :::*                   users:(("java",pid=3385,fd=254))
LISTEN     0      128         :::1883                    :::*                   users:(("java",pid=3385,fd=314))
LISTEN     0      50          :::44444                   :::*                   users:(("java",pid=3385,fd=245))

# grep activemq etc/system.properties 
activemq.port = 61616
activemq.host = localhost
activemq.url = tcp://${activemq.host}:${activemq.port}

```

Observe:

* JBoss AMQ is embeded in [karaf](https://karaf.apache.org/).
* Port 8181 is the [mnt console](https://access.redhat.com/documentation/en-us/red_hat_jboss_a-mq/6.3/html/management_console_user_guide/fmcug_introduction_accessing) and login with `admin/admin` by default. The Web UI looks more advanced.
* ActiveMQ is running with port 61616.

TODO:

* go-stomp is not working yet with JBoss AMQ 6.3.
* Which version of JBoss AMQ is the testing target? 7.2 vs 6.3
* Msg model (e2e vs pub/sub) and Protocol (many are supported, openwire,stomp ...)?
* Data store engine: [depending on 6.3 or 7.2, many supported, journal, jdbc ...](https://access.redhat.com/documentation/en-us/red_hat_amq/7.2/html/migrating_to_red_hat_amq_7/message_persistence)

## Template on OCP

```sh
# oc get template -n openshift | grep amq
```

## Benchmarks

* https://github.com/romankhar/IBM-MQ-vs-ActiveMQ-peformance-test
* https://github.com/hinunbi/a-mq-bmt
