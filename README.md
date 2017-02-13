# Trouble Maker
Proactive Failure
-----------------
A successful Microservices platform requires a durable and resilient environment that supports the ability to continuously deploy multiple services. Automated deployment is a must, and when possible, automated recovery from failures should be implemented, because failures will happen, (i.e. "Murphy's Law").

Instead of waiting for a failure to occur and seeing how durable and resilient your platform is, be proactive and make failure a `USE CASE` of your platform. If you know failures are occurring, yet pagers are not going off at 3 a.m. and the help desk is not being called, then you know your system is durable. 

What Is Trouble Maker? 
---------------------
Netflix implemented `Chaos Monkey` to randomly take down services during normal business hours. Trouble Maker does the same thing, but also provides an ad hoc console to produce common troublesome issues in your platform so you can test its durability on demand. 

####How Trouble Maker Works
Trouble Maker is a Java Spring Boot application that communicates with a a client service that has a small servlet registered with a Java API-based service application. By default, Trouble Maker accesses `Eureka` to discover services, and based upon a `cron` task, randomly selects a service to kill (i.e. shut down).

By default, when started, once per day Monday through Friday, a random service will be selected and killed. This option can be turned off, and when this occurs, can be configured. See configuration options section below. 

####Installing And Running

#####Dashboard WEB UI 

1. Git clone this repo.

2. Start the Java Spring Boot APP, or import into a JAVA IDE and start (JRE 1.8+ required). 

	java -jar jar/khs-trouble-maker.jar
 
A Eureka instances is required by default, see setting Eureka properties below. 
 
3. Open browser with the following URL: 

	http://localhost:9110
	
`Note:` By default, basic auth is implemented using Spring Security, you will prompted for user id password, here are the credentials.

	User id: user 
	password: see below 
	
Check your console and you should see the `generated password` generated by `Spring Boot`, as the the screen shot shows:

![](/img/console-default-console.png)
	

#### Client Configuration

If using Spring Boot, you can can simply include the starter project, see directions [here](https://github.com/in-the-keyhole/khs-spring-boot-troublemaker-starter)

Otherwise, you can use the platform agnostic client found [here](https://github.com/in-the-keyhole/khs-trouble-maker-client)

#### Dashboard
The Trouble Maker Dashboard has an event log and also allows services to be selected and killed on demand. It also invokes other troublemaking issues against these services. NOTE: The Dashboard defaults to and discovers services from a local instance of `Eureka`. 

![](/img/trouble-screen.png)

####Dashboard Trouble Options
From the dashboard a service can be selected and the following troubles applied: 

`KILL` - Terminate the service (i.e. system exit will be performed). Tests fail over and alert mechanisms.

`LOAD` - The selected service will be invoked with numerous blocking API calls. Blocking time and number of threads can be specified. This emulates how service acts under API load.

`MEMORY` - The selected service will consume memory until HEAP memory is met, then will block for a specified time. This can be used to emulate how system performs under low memory conditions.

`EXCEPTION` - The selected service will throw an exception. This tests the exception handling, logging, handling, and reporting mechanisms of the service.

####Configuration Options

These options are defined in the property file located at src/main/resources/META-INF/spring/trouble.properties

	###When to invoke trouble maker KILL service, default 2:00 pm Monday through Friday
	trouble.cron=0 0 14 * * MON-FRI

	### Access token that trouble maker client  
	trouble.token=abc123

	###Operation timeout in milliseconds, 0 means forever, default is 5 minutes
	trouble.timeout=300000

	###Threads to spawn when Blocking trouble is invoked, default is 200
	blocking.threads=200  

	###Trouble Service name, defaults to trouble.maker
	trouble.service.name = trouble.maker
	
	###use https when accessing client servlet api
	trouble.ssl=false

These properties can be set from the command line using VM argument. An example is shown below:

	java -jar khs-trouble-maker.jar -Dtrouble.timeout=200000

####Pluggable Service Registry

By default, Trouble Maker uses Netflix's Eureka service registry. However, any other service registry mechanism can be used.  Trouble Maker uses a service registry to discover services instances that it can apply trouble to.  

To use an alternative service registry, simply implement the Interface contract shown below: 

	public interface IServiceRegistry {
		public void start();
		public String lookup(String serviceName);	
		public List<String> serviceNames();
	}

Then register it in the `src/main/resources/META-INF/spring/application-context-service-registry.xml` file as shown below. 

	<bean id="serviceregistry" class="khs.trouble.service.impl.MyServiceRegistry">
	</bean> 

####Token-Based Access 

Requests made to a client trouble servlet supplies a token defined in the `trouble.properties` in the request header. This token needs to match the token supplied in the trouble client servlets init parameters. If they match then the operation will be performed. This type of access should be sufficient for internal behind-the-firewall access.

Here's a [LINK](https://github.com/in-the-keyhole/khs-trouble-maker-client) to the Trouble Client repository.  

####Eureka Registry Properties 

Trouble Maker is implemented to register itself with a Eureka registry. You can set Eureka client properties from the command line. Here's how a Eureka registry location is set from the command line:

	java -jar khs-trouble-maker.jar -Deureka.serviceUrl.default=http://localhost:8761/eureka/

You can also create a Jar and set properties in the `src/main/resources/eureka-client.properties` file. 
