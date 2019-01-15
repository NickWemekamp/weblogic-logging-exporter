# Weblogic Logging Exporter

An exporter that adds a log event handler to the weblogic server, such that WLS server Logs can be integrated to  [Elastic Stack](https://www.elastic.co/products) in Kubernetes directly,  by using the REST API in [Elasticsearch](https://www.elastic.co/products/elasticsearch).  

The blog [Let WebLogic work with Elastic Stack in Kubernetes](https://blogs.oracle.com/weblogicserver/let-weblogic-work-with-elk-in-kubernetes) outlines steps that can be used to harvest WLS server logs using Logstash so  that they can be filtered, manipulated, and viewed using Elasticsearch and Kibana. 
However, that approach requires the setup of a shared volume which is outside of the pod, and the logs needs to be written out to an intermediate log file for harvesting and parsing.

The goal of this project is to provide an easy to configure, robust and production ready solution to access WLS log information through Elasticsearch and Kibana.

## Getting Started

These instructions outlines the steps to build the _weblogic-logging-exporter.jar_ , which can then be integrated to the [Weblogic Kubernetes Operator](https://github.com/oracle/weblogic-kubernetes-operator/) for handling the Server logs generated by the Weblogic Server Domain. 

### Prerequisites

Weblogic Logging Exporter depends on the Weblogic Logging Jar which is available from the [ Oracle Maven Repository](http://maven.oracle.com/).

To access the Oracle Maven Repository, there are two fundamental requirements to be aware of:

1. You must be using [Maven 3.2.5](http://maven.apache.org/docs/3.2.5/release-notes.html) or later.  This contains the version of the component [Wagon 2.8](http://maven.apache.org/wagon/) that has been enhanced to support access to artifacts that are protected by [HTTP authentication schemes](https://issues.apache.org/jira/projects/WAGON/issues/WAGON-422).

2. You must be registered with OTN and have accepted the agreement to access and use the Oracle Maven Repository.  This can be done with either a new or an existing OTN user account by accessing the http://maven.oracle.com site and clicking the registration link.  
Once registered, you then just need to configure your local Maven environment with the details of the Oracle Maven Repository, including information that relates to the authentication model specifying your OTN username and password.

### Building
#### Configure Maven

**1. Encrypt your OTN Password**

You need to configure Maven to use the Oracle Maven Repository. This involves telling Maven about the repository and providing the necessary information to authenticate to the repository.
The following steps shows you how to save your OTN password in your _settings.xml_ file so  that you do not have to specify them manually every time.
Oracle recommends that you encrypt your password, using the utilities provided with Maven.

Here is the steps to encrypt your password and save that in your settings.   Refer to [Maven Password Encryption](http://maven.apache.org/guides/mini/guide-encryption.html) for more information.

```
mvn --encrypt-master-password my_master_password

```
The output will be a string similar to this:  

_{abcdefghijklmnopqrstuvwxyz123=}_ 

Save this value in your Maven _settings-security.xml_ file 

A copy of your _~/.m2/settings-security.xml_  will look like this
```
<settingsSecurity>
  <master>{abcdefghijklmnopqrstuvwxyz123=}</master>
</settingsSecurity>
```

Now that you have a master password defined, you can encrypt your OTN password using the following command:
```
mvn --encrypt-password my_OTN_password
```
The output will be another string that looks like this:  

_{==thisIs12My34_OTN_45_encrypted==}_

This will be the password that you provide in the _server_ section of your _settings.xml_ file when you configure the HTTP Wagon.  

**2. Adding the Oracle Maven Repository to Your settings.xml**

Add a repository definition to your Maven settings.xml file. The repository definition should look like the following:

``` 
<profiles>
    <profile>
      <id>main</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>maven.oracle.com</id>
          <url>https://maven.oracle.com</url>
          <layout>default</layout>
          <releases>
             <enabled>true</enabled>
          </releases>
        </repository>
      </repositories>

      <pluginRepositories>
        <pluginRepository>
          <id>maven.oracle.com</id>
          <url>https://maven.oracle.com</url>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
```

The Maven settings.xml requires additional settings to support the Oracle Maven Repository. 
Add the following _<server>_ element to the _<servers>_ section of the Maven settings.xml to configure the HTTP Wagon.
```
<servers>
    <server>
      <username>my_OTN_user_name</username>
      <password>{==thisIs12My34_OTN_45_encrypted==}</password>
      <configuration>
        <basicAuthScope>
          <host>ANY</host>
          <port>ANY</port>
          <realm>OAM 11g</realm>
        </basicAuthScope>
        <httpConfiguration>
          <all>
            <params>
              <property>
                <name>http.protocol.allow-circular-redirects</name>
                <value>%b,true</value>
              </property>
            </params>
          </all>
        </httpConfiguration>
      </configuration>
      <id>maven.oracle.com</id>
    </server>
  </servers>
```

If needed, you will need to specify the _proxies_ that is required. 

**3. Clone the project and build**
```
git clone git@orahub.oraclecorp.com:derek.sharpe/wls-logging-exporter.git
mvn install
```

The _weblogic-logging-exporter-1.0-SNAPSHOT.jar_ should be available under _target_ directory 

## Deployment

TBD.  Update on how to integrate this jar to weblogic kubernetes operator.

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.


## License

This project is licensed under the [_Oracle Universal Permissive License, Version 1.0_](http://oss.oracle.com/licenses/upl)