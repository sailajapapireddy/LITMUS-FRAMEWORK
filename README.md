<h1 align="center"> LITMUS 1.0</h1>
<p align="center">
    Java-based BDD Test Automation Framework
    <br/>
</p>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Contents</summary>
  <ol>
    <li><a href="#about-the-project">About the Project</a></li>
    <li><a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#folder-structure">Folder Structure</a></li>
      </ul>
    </li>
    <li><a href="#framework-installation-setup">Framework Installation & Setup</a>
      <ul>
        <li><a href="#pre-requisites">Pre-requisites</a></li>
        <li><a href="#setup">Setup</a></li>
        <li><a href="#self-healing">Self-Healing</a></li>
      </ul>
    </li>
    <li><a href="#writing-your-first-test-scenario">Writing your first test scenario</a> </li>
    <li><a href="#execution">Execution</a>
      <ul>
        <li><a href="#local-execution">Local Execution</a></li>
        <li><a href="#command-line-execution">Command-line Execution</a></li>
        <li><a href="#ci-cd-integration">CI/CD Integration</a></li>
      </ul>
    </li>
    <li><a href="#reports">Reports</a></li>
    <li><a href="#further-reading">Further Reading</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a> </li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>
<br/>

## About the project

This is a multipurpose kickstarter framework built on Java and Cucumber. The framework has BDD at its core
as it allows you to write tests in plain English using Gherkin syntax that are easy to understand.

```
Scenario Outline: Verify the login functionality
    Given user has opened the homepage in browser
    When user enters the <username> and <password>
    Then user should be successfully logged in
    Example:
       | username   | password  |
       | user1      | pwd123    |
```
This framework also has a rich pre-built library of utilities that lets you start developing test scripts and
executing those right from day 1.

## Getting Started

### Folder Structure

The framework uses the following folder structure for various script development and framework enhancement modules.
Please refer the structure to understand how the packages and files are organized.

```
pom.xml                             # For managing maven dependencies, build management and commandline arguments for runtime 
input-data/                         # All the test-data files created by testers should be put here    
infra/                          ┐
└───db/                         │
│   └───sql/                    ├   # These files are required for configuring and starting docker for Healenium
│       └───init.sql            │
└───docker-compose.yaml         ┘
src/
└───main/
│   └───java/                    
│   │   └───basetest/          ┐
│   │   └───constants/         ├    # Framework packages which are maintained by the CoE team
│   │   └───driver/            ┘
│   │   └───pageFactory/            # Add all your page classes in this package
│   │   │   └───PageClass           # Default page class which MUST BE extended by every page-object class in the test project
│   │   │   └───PageObjectManager   # For creating and managing objects of every page class.
│   │   └───utils/                  # Various utility libraries which can be used by both framework developers and testers
│   └───resources/
│       └───log4j2.xml              # Configuration for logging
└───test/                     
    └───java/
    │   └───runner/                  
    │   │   └───TestRunner          # Main Runner class for this cucumber framework. Used for running tests locally
    │   └───stepdefinitions/        # Add all your step-definition classes here
    │       └───CommonSteps         # A step-definition class for performing common actions such before and after test tasks etc.
    └───resources/
        └───Configs/                # All the project configurations are maintained here. Create as many required for each environment of your project
        │   └───qa.properties       
        │   └───dev.properties      
        └───Features/               # All feature files consisting of test scenarios are kept in this folder.
        └───cucumber.properties
        └───healenium.properties    # Self-healing can be enabled from here, along with other Healenium properties.
logs/                               # Metric and execution logs displayed here
target/                             # Execution results will be shown under this folder
└───cucumber-html-reports/          # HTML reports will be created under this folder
    └───overview-features.html      # Look for this file to open and view the execution reports.
README.md
Contributing.md

```

The folders that need to be maintained

- By automation testers

```
pom.xml
input-data/testdata.xlsx
src/test/java/stepdefinitions
src/test/resources/Configs
src/test/resources/Features
src/main/java/pageFactory
```

- By COE developers

```
pom.xml
src/main/java/basetest
src/main/java/driver
src/main/java/utils
src/main/java/pageFactory/PageClass.java
src/test/java/runner/TestRunner.java
```

## Framework Installation & Setup

### Pre-requisites

- Install [Java 1.8][java-1.8] or above
- Install [Maven][maven]
- Set Java and Maven in the [classpath][classpath]
- Install [Git][git] and clone this repository
- Install [IntelliJ Idea][intellij] (preferred) or any other compatible IDE
- Install all recommended [IntelliJ plugins][intellij-plugins]. Install the corresponding plugins if you're using any
  other IDE.
- Docker for Self-Healing _(optional)_: Healenium requires Docker for execution. If you wish to enable self-healing
features, make sure that Docker is installed in your test environment. 
<br/>Ignore this step, if you don't have docker setup in test environment, or do not wish to use self-heal features.
_[Install Docker Desktop on Windows][docker-install]_

### Setup

- Open the [qa.properties][qa-properties] file in src/test/resources/Configs folder and update the configuration details
  such as:
  <br/> Execution Server
  <br/> Grid details
  <br/> Test browser
  <br/> Application Base URL
- Open [pom.xml](pom.xml) and navigate to the plugin `org.apache.maven.plugins` under the `<build>` section. Update the
  name of your properties file in the `envName` variable.
  <br/> For instance, if your properties file name is qa.properties, then set the value of `envName` to _qa_.

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    ...
    <configuration>
        <systemPropertyVariables>
            <envName>qa</envName>
```

- If you wish to execute the test cases in any other environment such as dev or staging, then clone the qa.properties
  file, rename it appropriately, and update all the values corresponding to your new environment.
  Correspondingly, set the value of `envName` in pom.xml to the same name.
- For running your tests in [Headless Mode][headless-mode], set the value of `healessMode` to _true_.

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    ...
    <configuration>
        <systemPropertyVariables>
            ...
            <headlessMode>true</headlessMode>
```

### Self-Healing

To configure self-healing, set the properties in `src/test/resources/healenium.properties`
- **heal-enabled** is the flag to enable or disable healing. The accepted values are true and false
- **score-cap** is the score value to enable healing with predefined probability of match (0.5 means that healing 
will be performed for new healed locators where probability of match with target one is >=50% )
- **recovery-tries** is the number of times the algorithm will try to find a matching locator
- **hlm.server.url** is the ip:port or URL where hlm-backend instance is installed. _**Leave this value unchanged_
- **hlm.imitator.url** is the ip:port or URL where imitate instance is installed. _**Leave this value unchanged_

After enabling self-healing in `healenium.properties`, start the healenium-backend:
1. Make sure Docker Desktop is up and running in your test environment.
2. Start a CMD/Terminal in the Litmus project folder and run the following commands:
```
> cd infra
> docker-compose up -d
> cd..
```
3. After this, follow the instructions to <a href="#writing-your-first-test-scenario">write your test scenarios</a>
and <a href="#execution">execute</a>.
4. To stop the healenium-backend after your tests are executed, run the following commands:
```
> cd infra
> docker-compose down
> cd..
```

Self-healing requires that the tests are run at least once with the correct locators in order to set the baseline.
After the first run, healenium will be able to use its machine-learning algorithm to identify changes in locators in 
subsequent test runs.


## Writing your first test scenario

Once the framework installation and setup is complete, you are now ready to start writing your test scripts.

1. First create a new feature file in _src/test/resources/Features_ folder. Right-click on the Features folder and create a
  new file. Give it an appropriate name and make sure the file name ends with _.feature_. IntelliJ will automatically
  detect it as a cucumber feature file (given that you have installed all the plugins correctly).
2.Write your test scenarios in this feature file. Refer [Further Reading](#further-reading) to know more about 
writing test scenarios using Gherkin.
3. Then create a new Java class in _src/test/java/stepdefinitions_. Refer [Further Reading](#further-reading)
to know more about designing step-definitions for your Gherkin scenarios.
4. Make sure that you extend _TestClass_ in all your step-definition classes.
```
public class UserRegistrationSteps extends TestClass {
```
_These next steps are required only if you're performing Web automation testing._
4. Next create a new Java class in _src/main/java/com.sogeti.automation.test.pageFactory_. This class will be your page factory. You can create page objects and their corresponding methods in this class.
5. Make sure that you extend _PageClass_ in all your page classes.
```
public class UserRegistrationPage extends PageClass {
```
6. Create a constructor in your page class as shown below:
```
public UserRegistrationPage(WebDriver driver) {
    super(driver);
    wait = new WebDriverWait(driver, Duration.ofSeconds(FrameworkConstants.MediumWait));
    PageFactory.initElements(driver, this);
}
```
7. Next, open _PageObjectManager_ class and make an entry for your page class as shown below:
```
public class PageObjectManager {
  ...
  private UserRegistrationPage userRegistrationPage;
  ...
  ...
  public UserRegistrationPage getUserRegistrationPage() {
    return (userRegistrationPage == null) ? userRegistrationPage = new UserRegistrationPage(driver) : userRegistrationPage;
  }
```
Refer [Further Reading](#further-reading) to understand more about PageObjectManager.
8. Now go back to your step-definition class which you created in step 3, and create a constructor as shown below:
```
private UserRegistrationPage userRegistrationPage;
...
...
public UserRegistrationSteps(TestContext context) throws Exception {
    super();
    this.testContext = context;
    userRegistrationPage = testContext.getPageObjectManager().getUserRegistrationPage();
    ThreadContext.pop();
    ThreadContext.push(this.getClass().getSimpleName());
}
```
9. Assign tags to you test scenarios in the Feature files. Update the same tags in the _TestRunner_ class for execution.
```
tags = "@logintests",
```
Follow the above steps for all your feature files, step-definitions classes and page-object classes.

## Execution
### Local Execution
To execute your test cases locally in your IDE, right-click the _TestRunner_ class and select Run. Make sure that you have updated the _tags_ of the test(s) that you want to run in the TestRunner class.

### Command-line Execution
Open a command prompt and navigate to your project folder. Use the following maven command to execute your tests:
```
mvn clean verify -DenvName=qa -DheadlessMode=false -Dcucumber.filter.tags=@alltests
```
Let's understand the additional arguments in the above command.
- `-DenvName` will take the name of your property file. For instance, if your property file name is qa.properties, then set the value of `-DenvName` to _qa_.
If you wish to execute the test cases in any other environment such as _dev_ or _staging_, then set the value of `-DenvName` to the name of the corresponding property file.
- `-DheadlessMode` will take value as _true_ or _false_. If you wish to run your UI tests in headless mode, then set the value to _true_.
- `-Dcucumber.filter.tags` will take the tag-name of the test scenario(s) that you wish to execute.
<br/> Refer [Further Reading](#further-reading) to know more about maven command-line arguments.

_Note: If you do not pass the additional arguments with the maven command, 
then maven will take the default values of the System Property Variables that are set in pom.xml,
and the tags that are passed in TestRunner class. 
If no tags are specified in TestRunner, then maven will execute all the tests that are defined in the feature files._

### CI/CD Integration
LITMUS framework can be easily configured to run your tests on a CI/CD pipeline. 
The same maven command is used for executing the test scripts in CI/CD pipeline. Specify the below command in the 
build goals of your Jenkins setup or the pipeline execution script.
```
mvn clean verify -DenvName=qa -DheadlessMode=true
```
Ideally you should not pass any tags while executing on the pipeline, since the objective is to execute all your tests.
However, if you wish to execute a subset of test scripts, then add the `-Dcucumber.filter.tags` argument as well to the 
above command and pass the desired tag(s) as shown in [Command-line Execution](#command-line-execution) section.
<br/> It is also a good practice to run your UI tests in headless mode for faster execution.
<br/> _Note: If you don't have 
any UI tests in your test suite, then you can ignore the `-DheadlessMode` argument._


## Reports
Using LITMUS framework, you can generate pretty HTML reports with stats and charts showing the results of execution at 
Features, Test Scenario and Test Step levels. The generated report has no dependency so can be viewed offline.
<br/>
<img align="center" src=".blob/images/cucumber-html-report.png" alt="Cucumber HTML Report" style="width:600px"/>

Cucumber HTML Reports are generated under `target/cucumber-html-reports` folder. Look for the `overview-features.html` file,
simply double-click to open the report.
<br/> Cucumber HTML Report can be easily integrated with Jenkins as well. Please refer [Further Reading](#further-reading)
for more details.


## Further Reading
- Know more about [Cucumber Frameworks][cucumber-frameworks]
- Use of [Page Object Manager][page-object-manager] in a BDD framework
- Know more about [Gherkin][gherkin]
- How to write [Gherkin test scenarios][gherkin-test-scenarios]
- Learn to use [Tags][cucumber-tags] efficiently with cucumber test scenarios
- Writing [Step Definitions][step-definitions]
- Know more about [Web Driver Manager][webdrivermanager]
- Why should you run your tests in [Headless Mode][headless-mode]?
- Know more about [Maven command-line options][maven-arguments]
- [Parallel execution][parallel-execution] using Cucumber
- Know more about [Cucumber HTML Reports][cucumber-reporting]
- How to configure Cucumber HTML Report in [Jenkins][cucumber-report-jenkins]
- Learn more about [Healenium][healenium] and follow its [Github][healenium-github] page for documentation

## Roadmap
- [x] Base framework
    - [x] Environment management
    - [x] Test input management
    - [ ] Parallel execution
    - [x] Cucumber HTML report integration
- [x] Web Automation
    - [x] Web driver management
    - [x] Local execution
    - [x] Grid execution
    - [x] Multi-browser support
    - [x] Self-heal capabilities
    - [ ] Browserstack/Saucelabs integration
- [x] API Automation
    - [x] API authentication support
- [ ] Mobile Automation
    - [ ] Android - Browsers
    - [ ] Android - App
    - [ ] Android - Emulators
    - [ ] iOS - Browsers
    - [ ] iOS - App
    - [ ] iOS - Emulators
- [ ] Desktop app Automation
- [ ] CQA analytics extraction
- [ ] Analytics management
- [ ] Automated manual efforts estimator
- [ ] Applitools integration
- [ ] Add commonly used selenium operations into keywords
    - [ ] Dropdown management
    - [ ] List traversal and searching
    - [ ] List comparison
    - [ ] Random selection from dropdown
    - [ ] Random selection from checkboxes

## Contributing

To contribute to this repository, please see the [contribution guidelines](CONTRIBUTING.md).

## Contact

For further information, inquiries and support, please reach out to **QE&T Automation CoE** - [DL IN Sogeti Test Automation COE](sogetitestautomationcoe.in@capgemini.com).


<!-- reference urls -->

[git]: https://git-scm.com/
[java-1.8]: https://www.oracle.com/java/technologies/javase/javase8u211-later-archive-downloads.html
[maven]: https://maven.apache.org/install.html
[intellij]: https://www.jetbrains.com/idea/
[classpath]: https://docs.oracle.com/javase/tutorial/essential/environment/paths.html
[intellij-plugins]: ./.idea/plugins.json
[qa-properties]: ./src/test/resources/Configs/qa.properties
[cucumber-frameworks]: https://github.com/RameshGhk/Cucumber_Test_Automation_Framework
[page-object-manager]: https://www.toolsqa.com/selenium-cucumber-framework/page-object-manager/
[gherkin]: https://cucumber.io/docs/gherkin/reference/
[gherkin-test-scenarios]: https://cucumber.io/docs/guides/10-minute-tutorial/#write-a-scenario
[cucumber-tags]: https://cucumber.io/docs/cucumber/api/#tags
[step-definitions]: https://cucumber.io/docs/cucumber/step-definitions/
[parallel-execution]: https://cucumber.io/docs/guides/parallel-execution/
[cucumber-reporting]: https://github.com/damianszczepanik/cucumber-reporting
[cucumber-report-jenkins]: https://github.com/jenkinsci/cucumber-reports-plugin/wiki/Detailed-Configuration
[webdrivermanager]: https://bonigarcia.dev/webdrivermanager/
[headless-mode]: https://smartbear.com/blog/selenium-tests-headless/
[maven-arguments]: https://books.sonatype.com/mvnref-book/reference/running-sect-options.html
[docker-install]: https://docs.docker.com/desktop/install/windows-install/
[healenium]: https://www.automatetheplanet.com/healenium-self-healing-tests/
[healenium-github]: https://github.com/healenium/healenium-web