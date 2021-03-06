# APIT :: HOW-IT-WORKS
## API Testing Framework
### A Java Framework for Regression Testing of REST APIs
***

### What is at the heart of APIT?

The core of the APIT framework is the comparison of API responses that are created as Testcases. Each Testcase represents a unit test. The framework creates dynamic JUnit tests that are picked up by the JUnit platform for execution.

#### (A) JUnit Dynamic Tests: JSON Comparison
The ```expected``` responses are recorded in the APIT store before the refactoring. Then the tests are run. A test fails if the respective ```expected``` and ```actual``` responses differ from each other. Since API responses are in JSON (JavaScript object notation) format, the responses are compared at JSON level.

* **net.javacrumbs.jsonunit API**  
To compare ```expected``` and ```actual``` API responses as JSON strings.

* **javax.json API**  
As alternative to the above API, JSON strings can be transformed into JSON objects by leveraging the javax.json API. Consequently 2 JSON objects can be compared.

#### (B) GitDiff: A custom-made Diff tool
To further investigate on the actual differences of 2 JSON a git-based diff tool has been designed and implemented in APIT. This diff tool is an optional feature and can be enabled/disabled in the APIT properties before recording and the test run.

#### JUnit Dynamic Tests vs. GitDiff

This timeline visualizes the major difference between the 2 concepts (A) and (B).

![](./_images/graphic/color.png)

The ```expected``` API responses are 'acquired' and stored in the APIT ```/store``` before the test is run.
Each test-run brings 'new' ```actual``` API responses that overwrite the 'old' ```actual``` responses in the APIT ```/store```. Then the JUnit test compares ```actual``` with ```expected``` and fails when they differ. GitDiff on the other hand compares the ```actual``` of test(i) over time. For each test-run, when ```actual```  of test(i) differ from the ```actual``` the previous test(i) has run, a commit is made to the test-branch after that test-run.

***

### Framework Structure

<br />

#### 1. APIT Testclasses

<strong>Example: TestclassTest.java</strong>

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

<br />
<strong>1.1 Testclass.beforeAll()</strong>  

In ```TestGenerator.prepare(testclass)``` the ```/store``` is prepared.  
In case the GitDiff tool is enabled in the ```apit.properties``` file, the before part of GitDiff: ```GitDiff.runBefore()``` is run (blue color).


![](./_images/uml/Testclass.before.png)

<br />
<strong>1.2 Testclass.run()</strong>    

The JUnit Dynamic Tests are generated by the ```@TestFactory run()``` method, picked up by the JUnit 5 Platform and then executed by JUnit 5.
A single dynamic test fails, if ```expected``` API response differs from ```actual``` response.

![](./_images/uml/Testclass.run.png)

<br />
<strong>1.3 Testclass.afterAll()</strong>    

If the GitDiff tool is enabled in the ```apit.properties``` file, the after part of GitDiff: ```GitDiff.runAfter()``` is run (blue color).

![](./_images/uml/Testclass.after.png)

<br />

#### 2. APIT Testsuites  

<strong>Example: TestsuiteTest.java</strong>

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
<br />
<strong>2.1 Testsuite.beforeAll()</strong>   

Identical to *' **1.1 Testclass.beforeAll()**'* but with a minor twist.  
The scripts that are expected to run before the Testsuite: ```ScriptExecutor.runBatch(beforeScriptList)``` are executed first (blue color).

![](./_images/uml/Testsuite.before.png)

<br />
<strong>2.2 Testsuite.run()</strong>    

Identical to *' **1.2 Testclass.run()**'* .  

> It is worthwhile to mention that a Testclass runs 1 testclass ```.yml``` file.  
This is in contrast to a Testsuite that typically has 1 to n testclass ```.yml``` files to run.

![](./_images/uml/Testsuite.run.png)

<br />
<strong>2.3 Testsuite.afterAll()</strong>    

Identical to *' **1.3 Testclass.afterAll()**'* but with a minor twist.  
The scripts that are expected to run after the Testsuite: ```ScriptExecutor.runBatch(afterScriptList)``` are executed last (blue color).

![](./_images/uml/Testsuite.after.png)

***

### APIT Core Features

<br />

#### (A) JSON Comparison (JUnit Dynamic Tests)

<br />
<strong>JSON Comparison</strong>    

Remember that 1 to n Testcases are specified in a Testclass ```.yml``` file.  
The logic behind **'assertResponseBody'** and **'assertResponseStatus'** of each Testcase is responsible for triggering
* the Json comparison of response body by leveraging the ```net.javacrumbs.jsonunit.JsonAssert API```
* the value comparision of response status by leveraging the ```org.junit.Assert API```  

![](./_images/uml/assertJsonEquals-assertThat.png)


#### (B) GitDiff
<br />
<strong>GitDiff.runBefore()</strong>    

**beforeTest()**  
  * If the ```/store``` in the master-branch has changed,
    * add these changes to the stage - ```git add```
    * and committed to the master-branch - ```git commit```  


**prepareStore()**
  * checking out to the test-branch - ```git checkout```
    * merging any chances from the master-branch into the test-branch - ```git merge master-branch```
    * copying ```expected``` API responses into ```actual``` API responses
    * committing any changes to test-branch - ```git commit```
  * checkout to master-branch - ```git checkout master-branch```

![](./_images/uml/GitDiff.runBefore.png)

<br />
<strong>GitDiff.runAfter()</strong>    

**afterTest()**
  * (still on master-branch)
  * If the ```/store``` on the master-branch has changed,
    * add these changes to the stage - ```git add```
    * checkout to the test-branch - ```git checkout```
    * committing all changes to test-branch - ```git commit```
  * checkout to master-branch

![](./_images/uml/GitDiff.runAfter.png)
