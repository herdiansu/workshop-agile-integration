
## LAB 2 - Create Fuse REST Integration Project

Open JBoss Developer Studio application. Continue to work on fuse-rest project from lab1. If you haven't completed lab1, you can start with this project https://github.com/adithaha/jboss-fuse-workshop/raw/master/rest/solution/lab1/fuse-rest.zip and import into CodeReady Studio.

### fuse-rest preparation

Do this if you skip lab 1.
1. Download fuse-soap project https://github.com/adithaha/jboss-fuse-workshop/raw/master/rest/solution/lab1/fuse-rest.zip
2. Unzip fuse-rest.zip
3. Go to cmd, run command below to download all dependencies
```
cd fuse-rest
mvn clean package
```
4. Open CodeReady Studio
5. File - Import... - type 'maven' - Existing maven projects - Next - Browse... (select fuse-rest folder) - Finish

### LAB

1. Download wsdl file here https://raw.githubusercontent.com/adithaha/jboss-fuse-workshop/master/rest/employeeWS.wsdl, put into fuse-rest - src/main/resources  
2. Generate REST service from WSDL
```
- File - New - Other... - Camel REST From WSDL
	- WSDL file: put employeeWS.wsdl file location
	- Destination Project: fuse-rest
	- Next
	- Finish

Check if classes already generated in src/main/java - org.jboss.fuse.workshop.soap
Also check if camel route is generated in Camel Contexts - rest-springboot-context.xml
```

3. Configure API documentation, open fuse-rest - Camel Contexts - rest-springboot-context.xml - source. Replace all <restConfiguration> tag with below (line 9-10)
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
4. For easy configuration, put SOAP address on application.properties - src/main/resources - application.properties - source
```
...
url.employeeWS=http://localhost:8080/cxf/employeeWS

#url.employeeWS=${URL_EMPLOYEEWS}
#URL_EMPLOYEEWS=http://localhost:8080/cxf/employeeWS
...
```

5. Configure camel context to get SOAP address from properties Camel Contexts - rest-springboot-context. For each cxf endpoint, replace http://localhost:8080/cxf/employeeWS to {{url.employeeWS}}. For example:
```
Current Uri:cxf://http://localhost:8080/cxf/employeeWS?serviceClass=org.jboss.fuse.workshop.soap.EmployeeWSPortType&defaultOperationName=addEmployee
Change to Uri: cxf://{{url.employeeWS}}?serviceClass=org.jboss.fuse.workshop.soap.EmployeeWSPortType&defaultOperationName=addEmployee
```
There are 3 cxf endpoint that need to be changed.

6. Method with empty parameter is not configured correctly. getEmployeeAll service uri must be changed to not accepting parameter. rest-springboot-context.xml - tab REST - REST Operations - get /employeeall/{arg0}
From:
```
uri=/employeeall/{arg0
```
To:
```
uri=/employeeall
```

7. since getEmployeeAll doesn't have any parameter, its body need to be set to null before sending to soap backend. Go to design.
```
In route getEmployeeAll, insert below between _log3 and cxf
Transformation - Set Body
	- Expression: Simple
	- Expression: null
```
8. We will remove unnecessary headers. Go to design.
```
In route addEmployee, insert below between _to1 and _setHeader1
Transformation - Remove Headers
	- Pattern: *
```
```
In route getEmployee, insert below between _to2 and _setHeader2
Transformation - Remove Headers
	- Pattern: *
```
```
In route getEmployeeAll, insert below between _to3 and _setHeader3
Transformation - Remove Headers
	- Pattern: *
```
9. Configure Spring Boot to read generated camel xml - src/main/java - org.jboss.fuse.workshop.rest - Application.java
From:
```
@ImportResource({"classpath:spring/camel-context.xml"})
```
To:
```
@ImportResource({"classpath:spring/rest-springboot-context.xml"})
```
10. Remove generated maven settings file - configuration - settings.xml - right click - Delete - OK

11. Try your application
```
Build: right click your fuse-rest project - run as - maven build.. - Goal: clean package - Run
start fuse application: fuse-rest - src/main/java - org.jboss.fuse.workshop.rest - Application.java (right click) - run as - Java Application
```
Open browser, go to at http://localhost:8080/camel/api-docs. You should be able to see OpenAPI doc generated.

```
stop fuse application: go to console tab - click red square on the right
```
