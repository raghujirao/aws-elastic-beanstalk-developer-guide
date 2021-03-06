# Using the Elastic Beanstalk Tomcat platform<a name="java-tomcat-platform"></a>

The AWS Elastic Beanstalk Tomcat platform is a set of [environment configurations](https://docs.aws.amazon.com/elasticbeanstalk/latest/platforms/platforms-supported.html#platforms-supported.java) for Java web applications that can run in a Tomcat web container\. Each configuration corresponds to a major version of Tomcat, like *Java 8 with Tomcat 8*\.

Platform\-specific configuration options are available in the AWS Management Console for [modifying the configuration of a running environment](environment-configuration-methods-after.md)\. To avoid losing your environment's configuration when you terminate it, you can use [saved configurations](environment-configuration-savedconfig.md) to save your settings and later apply them to another environment\.

To save settings in your source code, you can include [configuration files](ebextensions.md)\. Settings in configuration files are applied every time you create an environment or deploy your application\. You can also use configuration files to install packages, run scripts, and perform other instance customization operations during deployments\.

The Elastic Beanstalk Tomcat platform includes a reverse proxy that forwards requests to your application\. The default server is [Apache HTTP Server Version 2\.4](https://httpd.apache.org/docs/2.4/)\. You can use [configuration options](#java-tomcat-namespaces) to configure Elastic Beanstalk to use the legacy [Apache HTTP Server Version 2\.2](https://httpd.apache.org/docs/2.2/) instead, or to use [nginx](https://www.nginx.com/)\. Elastic Beanstalk also provides configuration options to configure the proxy server to serve static assets from a folder in your source code to reduce the load on your application\. For advanced scenarios, you can [include your own `.conf` files](java-tomcat-proxy.md) in your source bundle to extend the Elastic Beanstalk proxy configuration or overwrite it completely\.

You must package Java applications in a web application archive \(WAR\) file with a specific structure\. For information on the required structure and how it relates to the structure of your project directory, see [Structuring your project folder](java-tomcat-platform-directorystructure.md)\.

To run multiple applications on the same web server, you can [bundle multiple WAR files](java-tomcat-multiple-war-files.md) into a single source bundle\. Each application in a multiple WAR source bundle runs at the root path \(`ROOT.war` runs at `myapp.elasticbeanstalk.com/`\) or at a path directly beneath it \(`app2.war` runs at `myapp.elasticbeanstalk.com/app2/`\), as determined by the name of the WAR\. In a single WAR source bundle, the application always runs at the root path\.

Settings applied in the AWS Management Console override the same settings in configuration files, if they exist\. This lets you have default settings in configuration files, and override them with environment\-specific settings in the console\. For more information about precedence, and other methods of changing settings, see [Configuration options](command-options.md)\.

## Configuring your Tomcat environment<a name="java-tomcat-options"></a>

The Elastic Beanstalk Tomcat platform provides a few platform\-specific options in addition to the standard options that all platforms have\. These options enable you to configure the Java virtual machine \(JVM\) that runs on your environment's web servers, and define system properties that provide information configuration strings to your application\.

You can use the Elastic Beanstalk console to enable log rotation to Amazon S3 and configure variables that your application can read from the environment\.

**To configure your Tomcat environment in the Elastic Beanstalk console**

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk), and then, in the regions drop\-down list, select your region\.

1. In the navigation pane, choose **Environments**, and then choose your environment's name on the list\.
**Note**  
If you have many environments, use the search bar to filter the environment list\.

1. In the navigation pane, choose **Configuration**\.

1. In the **Software** configuration category, choose **Edit**\.

### JVM container options<a name="java-tomcat-options-jvm"></a>

The heap size in the Java virtual machine \(JVM\) determines how many objects your application can create in memory before *[garbage collection](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/introduction.html)* occurs\. You can modify the **Initial JVM Heap Size \(\-Xms argument\)** and a **Maximum JVM Heap Size \(\-Xmx argument\)**\. A larger initial heap size allows more objects to be created before garbage collection occurs, but it also means that the garbage collector will take longer to compact the heap\. The maximum heap size specifies the maximum amount of memory the JVM can allocate when expanding the heap during heavy activity\.

**Note**  
The available memory depends on the Amazon EC2 instance type\. For more information about the EC2 instance types available for your Elastic Beanstalk environment, see [Instance Types](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) in the *Amazon Elastic Compute Cloud User Guide for Linux Instances*\. 

The *permanent generation* is a section of the JVM heap that stores class definitions and associated metadata\. To modify the size of the permanent generation, type the new size in the **Maximum JVM PermGen Size \(\-XX:MaxPermSize argument\)** field\. This setting applies only to Java 7 and earlier\.

### Log options<a name="java-tomcat-options-logs"></a>

The **Log Options** section has two settings:
+ **Instance profile** – Specifies the instance profile that has permission to access the Amazon S3 bucket associated with your application\.
+ **Enable log file rotation to Amazon S3** – Specifies whether log files for your application's Amazon EC2 instances should be copied to your Amazon S3 bucket associated with your application\.

### Static files<a name="java-tomcat-options-staticfiles"></a>

To improve performance, you can configure the proxy server to serve static files \(for example, HTML or images\) from a set of directories inside your web application\. When the proxy server receives a request for a file under the specified path, it serves the file directly instead of routing the request to your application\. You can set the virtual path and directory mappings in the **Static Files** section of the **Modify software** configuration page\.

For details about configuring static files using the Elastic Beanstalk console, see [Serving static files](environment-cfg-staticfiles.md)\.

### Environment properties<a name="java-tomcat-options-properties"></a>

In the **Environment Properties** section, you can specify environment configuration settings on the Amazon EC2 instances that are running your application\. Environment properties are passed in as key\-value pairs to the application\. 

The Tomcat platform defines a placeholder property named `JDBC_CONNECTION_STRING` for Tomcat environments for passing a connection string to an external database\.

**Note**  
If you attach an RDS DB instance to your environment, construct the JDBC connection string dynamically from the Amazon Relational Database Service \(Amazon RDS\) environment properties provided by Elastic Beanstalk\. Use JDBC\_CONNECTION\_STRING only for database instances that are not provisioned using Elastic Beanstalk\.  
For more information about using Amazon RDS with your Java application, see [Adding an Amazon RDS DB instance to your Java application environment](java-rds.md)\.

Inside the Tomcat environment running in Elastic Beanstalk, environment variables are accessible using the `System.getProperty()`\. For example, you could read a property named `API_ENDPOINT` to a variable with the following code\.

```
String endpoint = System.getProperty("API_ENDPOINT");
```

See [Environment properties and other software settings](environments-cfg-softwaresettings.md) for more information\.

## Tomcat configuration namespaces<a name="java-tomcat-namespaces"></a>

You can use a [configuration file](ebextensions.md) to set configuration options and perform other instance configuration tasks during deployments\. Configuration options can be defined by the Elastic Beanstalk service or the platform that you use and are organized into *namespaces*\.

The Tomcat platform supports options in the following namespaces, in addition to the [options supported for all Elastic Beanstalk environments](command-options-general.md):
+ `aws:elasticbeanstalk:container:tomcat:jvmoptions` – Modify JVM settings\. Options in this namespace correspond to options in the management console, as follows:
  + `Xms` – **JVM command line options**
  + `Xmx` – **JVM command line options**
  + `XX:MaxPermSize` – **Maximum JVM permanent generation size**
  + `JVM Options` – **JVM command line options**
+ `aws:elasticbeanstalk:environment:proxy` – Choose the proxy server and configure response compression\.
+ `aws:elasticbeanstalk:environment:proxy:staticfiles` – Configure the proxy to serve static assets from a path in your source bundle\.

The following example configuration file shows the use of the Tomcat\-specific configuration options\.

**Example \.ebextensions/tomcat\-settings\.config**  

```
option_settings:
  aws:elasticbeanstalk:container:tomcat:jvmoptions:
    Xms: 512m
    Xmx: 512m
    JVM Options: '-Xmn128m'
  aws:elasticbeanstalk:application:environment:
    API_ENDPOINT: mywebapi.zkpexsjtmd.us-west-2.elasticbeanstalk.com
  aws:elasticbeanstalk:environment:proxy:
    GzipCompression: 'true'
    ProxyServer: nginx
  aws:elasticbeanstalk:environment:proxy:staticfiles:
    /html: statichtml
    /images: staticimages
```

### Include Elastic Beanstalk configurations files<a name="java-tomcat-ebextensions"></a>

To deploy `.ebextensions` configuration files, include them in your application source\. For a single application, add your `.ebextensions` to a compressed WAR file by running the following command:

**Example**  

```
zip -ur your_application.war .ebextensions
```

For an application requiring multiple WAR files, see [Bundling multiple WAR files for Tomcat environments](java-tomcat-multiple-war-files.md) for further instructions\.

Elastic Beanstalk provides many configuration options for customizing your environment\. In addition to configuration files, you can also set configuration options using the console, saved configurations, the EB CLI, or the AWS CLI\. See [Configuration options](command-options.md) for more information\.