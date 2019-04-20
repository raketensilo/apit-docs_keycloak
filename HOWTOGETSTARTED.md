# APIT :: HOW-TO-GET-STARTED
## API Testing Framework
### A Java Framework for Regression Testing of REST APIs

***

### (A) The Problem
Developers refactor the business logic of a software with a specific goal in mind. With the refactoring they introduce changes to the system. Getting aware of all the effects (e.g. changed system outputs) that occur with these changes is crucial for developers. Otherwise the developers can introduce new bugs with the refactoring.

> Adjusting only a single or a few screws of a complex system can lead to multiple changes of the same system.  
Some of these changes (e.g. different system outputs) may be desired other changes might not.

### (B) A solution attempt: APIT
#### What is the basic idea?
> **1. Record** the **BEFORE state** of the system  
>
> **2. Refactor** the system  
>
> **3. Record** the **AFTER state** of the system  
>
> **4. Compare** the **EXPECTED (BEFORE) & ACTUAL (AFTER) state** of the system
>
> **5. Derive conclusions & further refactor** to the system
>
> **6. Repeat 1-5 until the refactoring goal is obtained**
>

#### How to 'record' and 'compare' the state of a system?
> **Record and Compare its API requests/responses**

***

### (C) Features of APIT
#### JSON COMPARISON : WITH JUNIT 5 DYNAMIC TESTS
**Test fails if 'expected' and 'actual' API requests/responses are different**  
JSON Strings or JSON objects can be compared
#### Testcases
A Testcase contains 1 or many requests (GET, POST)
#### Testclasses
A Testclass has and 1 or many Testcases and predefined settings (Testclass properties)
#### Testsuites
A Testsuite can group 1 or many Testclasses
#### Execute scripts before or after a test run
Can be specified for each Testsuite
#### Store & load complex API requests from file
complex POST requests can be stored in external files and are then loaded and sent to the API
#### Properties/settings with different scope
Define properties at global level (framework level) or define specific properties at testclass level
#### Support for Keycloak Authentication Management
Keycloak credentials can be set at global level (apit.properties) or testclass level (testclass properties)

***

### (D) How To: Use the APIT Framework

#### Manual Testing
In case you want to use APIT tests manually for your project inside your Java IDE (e.g. IntelliJ Idea).  
This can be achieved by clicking on a specific APIT test for your IDE to execute it.  
As an alternative you can run ```maven:test``` in your IDE to have the APIT tests executed all at-once.
>**1. Add the APIT framework to you project**  
>*see 'How To: Setup the Framework'*  
>
>**2. Create/Modify your APIT tests**  
>*see 'How To: Create an APIT test'*   
>
>**3. Clear the APIT store**  
>Make sure that there are no records for the Testclasses/Testsuites you are interested in  
>
>**4. Run your tests in Recording mode**  
>If the expected responses are not already in ```/store```,  
>they are retrieved from the API and save into the ```/store``` for comparison.  
>You should verify that the ```/store``` is populated accordingly.  
>
>**5. Make changes to API backend or API**  
>
>**6. Run your tests**  
>Each time APIT compares actual responses with the expected responses in ```/store```.  
>If actual and expected are different a test is expected to fail each time.  
>
>**7. Derive your conclusions**  
>By investigating your tests (respective Testcases, Testclasss, Testsuites)  
>The integrated diff tool in APIT can be of help
>
>**8. Make changes & Re-run tests (Optional)**  
> Repeat the following steps:  
>- **2-4 (Optional)**  
>- **5-8**  

#### Automated Testing
Before moving to automated testing it is advised to do manual testing of newly created APIT tests (*see 'Manual Testing'*).  
In case you want to run APIT tests for your project automatically the APIT tests can be executed all-at-once by Maven (build tool). Maven itself can be triggered by Jenkins (automation server).

[Apache Maven](https://git-scm.com/) (Maven) and [Jenkins Automation Server](https://jenkins.io/) (Jenkins) are out of the scope of this introduction. Plenty of tutorials about them can be found on the web.

***

### (E) How To: Deploy the APIT Framework
The simplest way to deploy APIT is to  
* build the APIT source code in your Java IDE with maven  
* Make sure to run the Maven build cycle until maven:install (at least)  

This will deploy new Maven package/dependency (a JAR file) locally into the ```/m2```-folder of your maven installation.  Additionally if you wish to deploy the package remotely you can upload the newly created Maven package (build artifact) to a build repository manager (e.g. Sonatype Nexus Repository Manager).

***

### (F) How To: Install & Setup the APIT Framework in your project

#### 1. Requirements

* Working API and API backend that you want to test
* If your API has access restriction:  
Working Keycloak authentication server and access credentials    
* Java 8 SE JDK
* Java IDE (e.g. IntelliJ Idea)
* Maven Dependency Manager
* Git

#### 2. Installation
After the steps in 'How To: Deploy the Framework' have been completed
* add the following dependency to the ```pom.xml``` (maven project settings) of the project you want to test   
* if the dependency/package is not in your local maven ```/m2``` directory but a remote repository you need to add the respective access credentials to the ```setting.xml``` in your local maven ```/m2``` directory to grant maven access to the remote dependency/package

**pom.xml**
```
<dependencies>
    ...
    <dependency>
        <groupId>com.abuscom</groupId>
        <artifactId>apit</artifactId>
        <version>1.0.2-SNAPSHOT</version>
        <scope>test</scope>
    </dependency>
    ...
</dependencies>
```
Finally make sure that Maven loads the newly added maven dependency/package to the ```pom.xml``` correctly.

#### 3. Setup
Global properties that shall be used in Testclasses and Testcases must be defined inside ```app.properties```.  
Therefore you should create a .properties file ```apit.properties``` in your project folder to set the essential global properties for the APIT framework.
##### Example: app.properties
For the purpose of better comprehension this exemplary ```app.properties``` file has been splitted into parts.
* **Basic info**
```
app.baseurl=http://localhost:9080/auth/
application.name=APIT
store.location=./src/test/resources/apit/store
```
* **Keycloak**  
Credentials for Keycloak support
```
keycloak.url=http://localhost:9080/auth/
keycloak.realm=master
keycloak.client=admin-cli
keycloak.granttype=password
keycloak.user=admin
keycloak.password=admin
```

#### 4. Structuring APIT files
* APIT Testclasses/suites shall be created in ```src/test/java/apit```

* Meta-data for the APIT Testclasses/suites shall be created under ```src/test/resources/apit``` in subdirectories ```/testclasses```,```/testsuites```,```/payload``` and ```/scripts```

* the APIT ```/store``` location according to exemplary ```apit.properties``` is ```src/test/resources/apit```

***

### (G) How To: Create an APIT Testsuite

##### Properties
A Testsuite
* is a group of Testclasses that are executed in bulk
* is NOT represented by a ```.yml``` file (only Testclasses are)

##### Features
A Testsuite
* offers the possibility to run scripts before or after the Testclasses
* scripts are to defined in ```src/test/apit/scripts```
* and then added to the 'before' or 'after' section in the desired Testsuite runner

***

### (H) How To: Run an APIT Testsuite
At the moment when a Testsuite is run, APIT stores all test recordings inside ```src/test/resources/store/${NAME_OF_TESTSUITE}```

* A Testsuite runner (e.g. *Testsuite1.java*) is created inside ```src/test/java```
* **Manual testing**  
the Testsuite runner can be executed manually inside your Java IDE
* **Automated testing**  
Testsuite runners can be triggered by **Maven** that itself can be evoked by automation server **Jenkins**.

A Testsuite runner
* extends **AbstractTestsuite**
* specifies the Testclasses (```.yml``` file) to be run
* calls **before()**, **generateTests()**, **afterAll()** provided by AbstractSuiteclass
* can specify scripts that are execute before or after the Testclasses

#### Example: TestsuiteTest.java
Here is an exemplary Testsuite that has been split into a sequence of Testclasses (```.yml``` files).

```
public class TestsuiteTest extends AbstractTestsuite {

    private static final List<String> before = Arrays.asList(new String[] {
            "./src/test/resources/apit/scripts/before.bat"
    });
    private static final List<String> testclasses = Arrays.asList(new String[] {
            "./src/test/resources/apit/testclasses/realms.testclass.yml",
            "./src/test/resources/apit/testclasses/roles.testclass.yml",
            "./src/test/resources/apit/testclasses/users.testclass.yml",
    });
    private static final List<String> after = Arrays.asList(new String[] {
            "./src/test/resources/apit/scripts/after.bat"
    });

    private static final String name = new Object() {}.getClass().getEnclosingClass().getSimpleName();


    @BeforeAll
    public static void beforeAll() {
        before(name, before, testclasses);
    }

    @Tag("APIT")
    @TestFactory()
    public List<DynamicTest> run() {
        return generateTests(name, testclasses);
    }

    @AfterAll
    public static void afterAll() {
        after(after);
    }

}
```
 <strong>1. beforeAll()</strong>
 * The scripts `before.bat` are executed.  
 * Tests are prepared: 'acquiring' `expected` API responses for all tests   

<strong>2. run()</strong>  
run the JUnit Dynamic Tests  

<strong>3. afterAll()</strong>  
* The script ```after.bat``` is executed.
***

### (I) How To: Create an APIT Testclass
##### Properties
A Testclass   
* extends ```AbstractTestclass``` (provided by the APIT Framework)
* is created inside ```src/test/java/apit```
* is composed of 1 ```.yml``` file located in ```src/test/java/apit/testclasses```
* Tests are created in ```src/test/resources```  
* Payloads (Json) can be added to API requests and are located in ```src/test/resources/payload```  

#### Example: realmsRolesUsers.testclass.yml
```
name: "realmsRolesUsers"
description: "get list of realms, roles and users"

properties:
  app.baseurl: "${app.baseurl}"
  store.location: "${store.location}"
  application.name: "${application.name}"
  keycloak.url: "${keycloak.url}"
  keycloak.realm: "${keycloak.realm}"
  keycloak.client: "${keycloak.client}"
  keycloak.granttype: "${keycloak.granttype}"
  keycloak.user: "${keycloak.user}"
  keycloak.password: "${keycloak.password}"

testcases:
  - name: "realmsRolesUsersTest1"
    authenticator: "keycloak"
    requests:
      - get: "admin/realms"
        assertResponseStatus: 200
        assertResponseBody: true
      - get: "admin/realms/${keycloak.realm}/roles"
        assertResponseStatus: 200
        assertResponseBody: true
      - get: "admin/realms/${keycloak.realm}/users"
        assertResponseStatus: 200
        assertResponseBody: true
```

***

### (J) How To: Run an APIT Testclass
When a Testclass is run, APIT stores all test recordings inside ```src/test/resources/store```

* The Testclass itself is defined as a ```.yml``` file that is then called by the respective Testclass runner
* A Testclass runner (e.g. *TestclassTest.java* or *TestsuiteTest.java*) is created inside ```src/test/java```
* **Manual testing**  
the Testclass runner can be executed manually inside your Java IDE
* **Automated testing**  
Testclass runners can be triggered by **Maven** that itself can be evoked by automation server **Jenkins** or similar.

A Testclass runner
* extends **AbstractTestclass**
* specifies the Testclass (```.yml``` file) to be run
* calls **before()**, **generateTests()**, **afterAll()** provided by AbstractTestclass

#### Example: TestclassTest.java

```
public class TestclassTest extends AbstractTestclass {

    private static final String testclass = "./src/test/resources/apit/testclasses/realmsRolesUsers.testclass.yml";

    private static final String name = new Object() {}.getClass().getEnclosingClass().getSimpleName();

    @BeforeAll
    public static void beforeAll() {
        before(name, testclass);
    }

    @Tag("APIT")
    @TestFactory()
    public List<DynamicTest> run() {
        return generateTests(name, testclass);
    }

    @AfterAll
    public static void afterAll() {
        after();
    }

}
```

***

### (K) How To: Create an APIT Testcase

##### Properties
A Testcase
- is part of a Testclass
- A Testcase among other Testcases form a Testclass (s. 'How To: Create an APIT Testclass')

##### Features
- can perform 1 or many API GET/POST requests
- large/complex request payloads (Json-Format) can be loaded from external files located in ```src/test/apit/payload/```

##### Example
Testcase **realmsRolesUsersTest1** : defined in **src/test/resources/apit/testclasses/realmsRolesUsers.testclass.yml**
```
...
testcases:
...
  - name: "realmsRolesUsersTest1"
    authenticator: "keycloak"
    requests:
      - get: "admin/realms"
        assertResponseStatus: 200
        assertResponseBody: true
      - get: "admin/realms/${keycloak.realm}/roles"
        assertResponseStatus: 200
        assertResponseBody: true
      - get: "admin/realms/${keycloak.realm}/users"
        assertResponseStatus: 200
        assertResponseBody: true
...    
```

***

### (L) How To: Run an APIT Testcase
A Testcase is run as part of a Testclass.  
See *'How To: Run an APIT Testclass'* for more details.

***

## Test Results & Conclusions

Like in typical Unit testing (JUnit) fashion:
>**When no error occurs the test passes successfully**

When the test fails with errors (or is not completed):
>**When the test fails a generic error information is prompted to the console**

To get more details about the failed test:
>**For each test ```_expected.json``` & ```_actual.json``` files can be found in the APIT ```/store``` directory to further investigate why a particular test has failed.**

***

## Built With

#### JUnit 5
* [JUnit 5](https://junit.org/junit5/) - Unit Testing Framework for Java
* [net.javacrumbs.jsonunit](https://github.com/lukas-krecan/JsonUnit) - Handy JSON library to use with JUnit

#### JSON
* [javax.json](https://docs.oracle.com/javaee/7/api/javax/json/package-summary.html) - Powerful JSON API for Java

#### Git
* [Git](https://git-scm.com/) - A free Version Control Software
* [Sourcetree](https://www.sourcetreeapp.com/) - A free Git GUI Client

#### Java 8
* [Java 8 SE JDK](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) - Java 8 Standard Edition Java Development Kit

#### Maven
* [Apache Maven](https://maven.apache.org/) - Dependency Management And Build automation tool for Java
