
## LAB 1 - Create Fuse REST Integration Project

Open JBoss Developer Studio application. Download wsdl file here https://raw.githubusercontent.com/adithaha/jboss-fuse-workshop/master/rest/employeeWS.wsdl

1. File - New - Project.. - type 'fuse' - Fuse Integration Project - Next
 ```
Project-name: fuse-rest
Next
Deployment Platform: Standalone
Runtime Environment: Spring Boot
Camel version: 2.21.0.fuse-731003-redhat-00003 (Fuse 7.3.1 GA)
Next
Simple log using Spring Boot - Spring DSL
Finish
Open Associate Perspective: Yes
 ```
2. Change groupId in pom.xml - fuse-rest - pom.xml line #4
 ``` 
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.jboss.fuse.workshop</groupId>
  <artifactId>fuse-rest</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <name>Workshop:: Fuse REST</name>
  <description>Workshop:: Fuse REST</description>
  ```
  Add few dependencies below:
   ``` 
  <dependencies>
    ...
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-cxf-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-servlet-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-jaxb-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-jackson-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-swagger-java-starter</artifactId>
    </dependency>
  </dependencies>
  
  ```

3. Change default package src/main/java - org.mycompany - right click - refactor - rename
	- New Name: org.jboss.fuse.workshop.rest
	- OK
	- Continue
4. Remove default route src/main/resources - spring - camel-context.xml
5. Generate REST service from WSDL
```
- File - New - Other... - Camel REST From WSDL
	- WSDL file: put wsdl file from employeeWS
	- Destination Project: fuse-rest
	- Next
	- Finish

Check if classes already generated in src/main/java - org.jboss.fuse.workshop.soap
```

6. Configure API documentation, replace <restConfiguration> tag in rest-springboot-context.xml with below (use source)

```
	<restConfiguration apiContextPath="api-docs" bindingMode="json"
            component="servlet" contextPath="/camel" enableCORS="true">
            <dataFormatProperty key="prettyPrint" value="true"/>
            <apiProperty key="cors" value="true"/>
            <apiProperty key="api.version" value="1.0.0"/>
            <apiProperty key="api.title" value="Red Hat Fuse - REST"/>
            <apiProperty key="api.description" value="Red Hat Fuse - REST"/>
            <apiProperty key="api.contact.name" value="Red Hat"/>
        </restConfiguration>
        
```
7. For easy configuration, put SOAP address on application.properties - src/main/resources - application.properties - Finish - source
```
...
url.employeeWS=${URL_EMPLOYEEWS}
URL_EMPLOYEEWS=http://localhost:8080/cxf/employeeWS
...
```

8. Configure camel context to get SOAP address from properties - rest-springboot-context. For each cxf endpoint, replace http://localhost:8080/cxf/employeeWS to {{url.employeeWS}} (use design)
```
Uri: cxf://{{url.employeeWS}}...
```

9. Method with empty parameter is not configured correctly. getEmployeeAll service uri must be changed in <rest> tag (use source)
From:
```
uri="/employeeall/{arg0}">
```
To:
```
uri="/employeeall">
```

10. since getEmployeeAll doesn't have any parameter, its body need to be set to null before sending to soap backend. Go to design.
```
In route getEmployeeAll, insert below between _log3 and cxf
Transformation - Set Body
	- Expression: Simple
	- Expression: null
```

11. Configure Spring Boot to read generated camel xml - src/main/java - org.jboss.fuse.workshop.rest - Application.java
```
@ImportResource({"classpath:spring/rest-springboot-context.xml"})
```


12. Try your application
```
Clean build: right click your fuse-rest project - run as - maven clean
Build: right click your fuse-rest project - run as - maven build....
	Goals: clean package
	Run
start fuse application: fuse-rest - src/main/java - org.jboss.fuse.workshop.rest - Application.java (right click) - run as - Java Application
```
Open browser, go to at http://localhost:8080/camel/api-docs

```
stop fuse application: go to console tab - click red square on the right
```