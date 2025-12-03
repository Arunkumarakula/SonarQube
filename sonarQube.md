# **In security testing we have two main approaches: SAST and DAST**

* **SAST** is Static Application Security Testing. It analyzes the source code or binaries during the build stage, before the application runs. In DevOps
  pipelines we use SAST to catch issues early like insecure functions, hardcoded secrets, and data-flow vulnerabilities.

  **Example tool:** SonarQube, Trivy

* **DAST** is Dynamic Application Security Testing. It tests the application from the outside once its deployed in a running environment, usually in staging. It
  helps identify runtime issues such as authentication flaws, misconfigurations, XSS, and injection vulnerabilities.

  **Example tool:** OWASP ZAP

---

# **SonarQube:**

SonarQube is a code quality and security analysis tool that automatically scans the source code to detect bugs, code smells, vulnerabilities,
duplications, and security issues. It also checks unit test coverage and Quality Gates to ensure only clean and consistent code.

---

# **SonarQube mainly performs three important functions in a DevOps pipeline**

## **1. Code Quality Analysis:**

* **Bugs:** These are coding mistakes that can cause your application to fail during runtime.
  Example: Null pointer exceptions, incorrect logic, errors in conditions.

* **Code Smells:** These are not bugs but bad coding practices that make the code hard to read, maintain, or update.
  Example: Long methods, unused variables, complex logic.

* **Duplicate Code:** SonarQube detects if the same code is repeated in different places. Duplicate code increases technical debt and makes maintenance
  harder.

* **Security Vulnerabilities:** These are actual weaknesses that attackers can exploit.
  Example: SQL injection, weak encryption, hardcoded passwords.

* **Security Hotspots:** A Security Hotspot is a part of the code that deals with sensitive operations and needs a manual review by developer or security
  engineer to confirm whether it is secure or not.

---

## **2. Code Coverage (Unit Testing Analysis):**

* SonarQube measures how much of the code is covered by unit tests. It shows tested lines, critical areas covered, and overall coverage percentage
  helping ensure the application is reliable and reducing deployment risks.

---

## **3. Quality gate check:**

* A Quality Gate is a set of rules or conditions that code must meet before it can be released or merged.
  Typical conditions include:

  * No new critical or major bugs
  * No new security vulnerabilities
  * Code coverage above a certain threshold
  * Maintainability and reliability ratings within limits
  * Duplication percentage below threshold

  If the code does not meet the Quality Gate criteria, the pipeline fails, preventing poor-quality code from moving forward.

---

# **SonarQube consists of three main parts:**

The SonarQube Server (dashboard + rules engine), the Sonar Scanner (which analyzes code and sends results), and a PostgreSQL database (to store all scan results and metadata).

## **1. SonarQube server:**

SonarQube Server is the main server that processes code analysis reports and displays results on a web dashboard.

* It handles project management, rule sets, quality gates, and provides the user interface.

## **2. sonar scanner:**

Sonar Scanner is the tool that scans the source code, generates the analysis report, and sends it to the SonarQube Server.

* It performs the actual code inspection for bugs, vulnerabilities, code smells, and coverage.

## **3. PostgreSQL:**

PostgreSQL is the backend database used by SonarQube to store all analysis data, project information, user details, and historical metrics.

---

# **Versions of SonarQube:**

## **1. Community Edition:**

* Community Edition is the free, open-source version of SonarQube that provides basic code quality and security analysis for multiple programming
  languages. It includes core features like bugs, code smells, duplications, vulnerabilities, and quality gates.

## **2. Developer Edition:**

* Developer Edition is a paid version that adds advanced security analysis, deeper programming language support, branch analysis, pull request
  decoration, and more detailed reports. It is designed for professional development teams.

## **3. Enterprise Edition:**

* Enterprise Edition is a full-featured, large-scale version designed for organizations with many projects. It includes portfolio management, advanced
  governance, scalability features, and enterprise-level reporting. It supports large teams and complex DevOps environments.

---

# **SonarQube Integration with CI/CD - Jenkins:**

## **1. Automate Code Quality Checks in CI/CD:**

Integrating SonarQube with Jenkins ensures that code quality and security analysis happens automatically every time developers push changes.

## **2. Catch Issues Early (Shift-Left Security):**

Bugs, vulnerabilities, code smells, and duplications are detected during the build stage, not after deployment.

## **3. Enforce Quality Gates:**

Jenkins pipeline can be configured to fail the build if SonarQube finds like Critical vulnerabilities, Low code coverage, Too
many code smells, Duplications. This prevents bad code from reaching staging or production.

---

# **How SonarQube Integration Works with Jenkins:**

* Install the SonarQube Plugin in Jenkins
  eg: SonarQube Scanner - This allows Jenkins to communicate with the SonarQube server.

* Configure SonarQube Server in Jenkins (**Manage Jenkins → Configure System → SonarQube Servers**).
  This tells Jenkins where to send the scan reports.

  Add:

  * SonarQube server URL: [http://3.7.49.8:9000](http://3.7.49.8:9000)
  * Authentication token
  * Server name: SonarQube

* Install & Configure Sonar Scanner (Manage Jenkins → Global Tool Configuration)
  Add SonarScanner installation - This is the tool that performs the code scan.

---

## **Add SonarQube Steps into Jenkins Pipeline**

```groovy
pipeline {
    agent any

    stages {
        stage('SonarQube Scan') {                                         
            steps {
                withSonarQubeEnv('MySonarServer') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
```

* sonar-scanner analyzes the code
* Results go to the SonarQube server
* Jenkins waits for Quality Gate result
* If Quality Gate fails → Jenkins build fails automatically
* Visualize Results in SonarQube Dashboard

**Interview Tip:**
We integrate SonarQube with Jenkins to automate code quality and security checks in CI/CD. Jenkins triggers SonarScanner during the build,
sends the results to SonarQube, and waits for the Quality Gate status. If the Quality Gate fails, Jenkins fails the pipeline. This ensures
only clean and secure code goes to deployment.

---

# **sonar-project.properties:**

The sonar-project.properties file is a configuration file used by the Sonar Scanner to understand how to analyze a project. It
defines key settings such as the project key, project name, source code paths, exclusions, and code coverage report locations.
The scanner reads this file during execution and uses these properties to send the correct analysis data to the SonarQube
server.

```ini
sonar.projectKey=admin-service
sonar.projectName=admin-service
sonar.projectVersion=1.0
sonar.sources=src
sonar.language=java
sonar.java.binaries=target/classes
sonar.sourceEncoding=UTF-8

# Export JSON report
sonar.report.export.path=sonar-report.json

# (Optional) for more detailed report use SARIF format
# sonar.sarifReportPaths=sonar-report.sarif.json
```

---

# **Sonar Token:**

A SonarQube Token is a secure authentication key used to connect external tools like Jenkins, GitHub Actions or SonarScanner to the SonarQube
server.

* Generated in SonarQube UI (**My Account → Security → Generate Token**)
* Save token and add this token in Jenkins credentials
* Manage Jenkins → Configure System

  * Name: MySonar
  * Server URL: http://<your-sonarqube-server>:9000
  * Authentication Token: Add the token you generated earlier

---

# **Sonar Reports:**

* By default SonarQube generates the sonar reports in JSON format we can download from the Extract.

* Another way is convert JSON format to the `.txt` format using the `jq -r` command in the CI/CD pipelines.

**Interview Tip:**
I generate reports from SonarQube by using its REST APIs. After the SonarScanner completes the analysis, I call APIs like `/api/issues/search`,
`/api/measures/component`, and `/api/qualitygates/project_status` using curl. The output is in JSON format, so I use `jq -r` to convert the JSON
into clean, human-readable `.txt` report files. These reports include issues, code quality metrics, and Quality Gate status. Finally, I archive
the `.txt` files in Jenkins so the team can download and review them.

---

# **Jacaco:**

JaCoCo generates the test coverage report for Java projects. It tracks which lines, methods, and branches are executed during unit tests. SonarQube itself cannot calculate coverage it only reads the JaCoCo XML report. So without JaCoCo, SonarQube would show 0% coverage, and Quality Gates related to coverage would fail.

---

# **Sonar Quality gate is taking much time because:**

SonarQube background tasks are taking a long time due to:

* SonarQube server is slow
* PostgreSQL database is slow
* Your project is very big
* Network is slow between Jenkins → SonarQube

