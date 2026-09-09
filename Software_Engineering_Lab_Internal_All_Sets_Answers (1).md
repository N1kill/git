# Software Engineering Lab Internal I — Complete Answers
## Maven, Git/GitHub and Docker

> Prepared from the uploaded KMIT Lab Internal question papers (CSM-C Sets 1–3 and CSM-A Set 1/3) and the specified GitHub repositories.
>
> **Important repository note:** I checked the specified `deepthisagar7/library-management` repository. Its current `pom.xml` is malformed and has Java compiler values of `1`; the repository also contains `LibraryManagement.java` with a `main()` method. The GitHub connection available here has **read-only permission**, so I could inspect the repository but could not push the correction directly.

---

# SET-3 — Library Management System
Repository: `https://github.com/deepthisagar7/library-management.git`

## Part I — Maven Java Application Development

### 1.a) Clone and import as Maven project

```bash
git clone https://github.com/deepthisagar7/library-management.git
cd library-management
```

### Eclipse
1. Open Eclipse.
2. Select **File → Import**.
3. Select **Maven → Existing Maven Projects**.
4. Select the cloned `library-management` directory.
5. Select `pom.xml`.
6. Click **Finish**.
7. Configure the project/JDK to Java 17.

### IntelliJ IDEA
1. Open IntelliJ IDEA.
2. Select **Open**.
3. Select the cloned `library-management` directory or `pom.xml`.
4. IntelliJ detects it as a Maven project.
5. Set the Project SDK to **JDK 17**.
6. Reload the Maven project.

---

## 1.b) Corrected `pom.xml` for Java 17 and executable JAR

The repository currently has malformed XML such as `<modelVersion>4.0.0<modelVersion>`, `<version>1.0-SNAPSHOT<version>`, and `<properties>` is not closed correctly. Its compiler source/target are also not set to Java 17.

A suitable corrected `pom.xml` is:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.library</groupId>
    <artifactId>library-management</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <build>
        <finalName>library-management</finalName>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.4.2</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.library.LibraryManagement</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

The `maven-jar-plugin` configuration makes the JAR executable because the repository's Java class contains:

```java
public static void main(String[] args)
```

After:

```bash
mvn clean package
```

the expected artifact is:

```text
target/library-management.jar
```

---

## 1.c) Maven lifecycle phases

### `clean`
Deletes the previous Maven build output, normally the `target/` directory.

### `compile`
Compiles the main Java source files.

### `test`
Compiles and executes the test cases.

### `package`
Packages the compiled application into the configured artifact, such as a JAR or WAR.

### `install`
Installs the generated artifact into the local Maven repository, normally:

```text
~/.m2/repository
```

### Which phase generates the JAR?

**`package`** generates the JAR.

---

## 1.d) Maven command

```bash
mvn clean package
```

---

## 1.e) Unsupported Java source/target version

First check Java installed on the machine:

```bash
java -version
javac -version
```

Check which Java Maven is actually using:

```bash
mvn -version
```

`mvn -version` displays the Java version and Java home Maven is using.

If Maven is using Java 8, configure Java 17.

On Windows, check:

```cmd
echo %JAVA_HOME%
where java
```

Set `JAVA_HOME` to the JDK 17 installation and ensure `%JAVA_HOME%\bin` is on `PATH`.

Then open a new terminal and verify:

```bash
java -version
mvn -version
```

The Maven compiler configuration should contain:

```xml
<maven.compiler.source>17</maven.compiler.source>
<maven.compiler.target>17</maven.compiler.target>
```

Then rebuild:

```bash
mvn clean package
```

---

## 1.f) JAR builds but does not execute — three possible causes

### 1. No `Main-Class` in the JAR manifest

Check:

```bash
jar tf target/library-management.jar
```

Inspect the manifest:

```bash
unzip -p target/library-management.jar META-INF/MANIFEST.MF
```

It should contain:

```text
Main-Class: com.library.LibraryManagement
```

Fix it with `maven-jar-plugin`.

### 2. Wrong main-class name

The class/package must match:

```text
com.library.LibraryManagement
```

Check:

```bash
jar tf target/library-management.jar
```

Then run:

```bash
java -jar target/library-management.jar
```

### 3. Java version mismatch

Check:

```bash
java -version
```

If the JAR was compiled for Java 17, run it with a compatible JRE/JDK.

Also inspect the class if required:

```bash
javap -verbose target/classes/com/library/LibraryManagement.class
```

---

## 1.g) Java 17 mismatch between developers

### 1. Identify mismatch

Run on both machines:

```bash
java -version
javac -version
mvn -version
```

Compare the displayed Java version and Java home.

### 2. Configure Maven to require Java 17

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>
```

The Maven compiler plugin can also explicitly use:

```xml
<configuration>
    <source>17</source>
    <target>17</target>
</configuration>
```

### 3. Why use the same JDK?

Using the same JDK version avoids:

- source/target compatibility errors,
- different compiler behaviour,
- different library/runtime compatibility,
- different generated bytecode,
- deployment failures.

---

# Part II — Git & GitHub

## 2.a) Initialize and push

```bash
git init
git status
git add .
git commit -m "Initial commit"

git remote add origin https://github.com/deepthisagar7/library-management.git

git branch -M main
git push -u origin main
```

---

## 2.b) Correct the last commit message

Since it has not been pushed:

```bash
git commit --amend -m "Added Book Management Module"
```

---

## 2.c) Create and switch to branch

```bash
git switch -c feature/book-management
```

Alternative:

```bash
git checkout -b feature/book-management
```

---

## 2.d) Restore accidentally deleted `Book.java`

```bash
git restore Book.java
```

If it is in a directory:

```bash
git restore path/to/Book.java
```

---

## 2.e) Resolve merge conflict

### 1. Identify conflicted files

```bash
git status
```

Git will list the files under **Unmerged paths**.

### 2. Open and resolve

Open the conflicted file. Git marks conflicts like:

```text
<<<<<<< HEAD
code from current branch
=======
code from other branch
>>>>>>> feature/book-management
```

Keep the required code and remove the conflict markers.

### 3. Stage the resolved file

```bash
git add path/to/conflicted-file
```

### 4. Complete the merge

```bash
git commit -m "Resolve merge conflict in book management"
```

### 5. Push

```bash
git push origin feature/book-management
```

If the merge was performed on `main`, push:

```bash
git push origin main
```

---

## 2.f) Remove unwanted staged files

To unstage everything without deleting files:

```bash
git restore --staged .
```

To unstage one file:

```bash
git restore --staged filename
```

Check staging:

```bash
git status
```

or:

```bash
git diff --cached
```

### Suitable `.gitignore`

```gitignore
# Maven
target/

# Eclipse
.classpath
.project
.settings/

# IntelliJ
.idea/
*.iml

# Environment files
.env

# Logs
*.log

# OS files
.DS_Store
Thumbs.db
```

After adding `.gitignore`:

```bash
git add .gitignore
```

If an unwanted file was already tracked, remove it from Git tracking while keeping it locally:

```bash
git rm --cached filename
```

For a directory:

```bash
git rm -r --cached directory/
```

Then:

```bash
git commit -m "Update gitignore and remove unwanted files"
```

---

## 2.g) Incorrect implementation in latest commit

### Commit NOT pushed

Correct the implementation and amend the latest commit:

```bash
git add .
git commit --amend
```

Or:

```bash
git commit --amend -m "Correct Book Management implementation"
```

Use this when the commit is local and has not been shared.

### Commit ALREADY pushed

Use `git revert`:

```bash
git revert <commit-hash>
git push origin main
```

This creates a new commit that reverses the unwanted changes.

**Do not normally use `git reset --hard` + force push on shared history.**

---

# Part III — Dockerization

## 3.a) Dockerfile

For this standalone Java 17 JAR application:

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY target/library-management.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

Note: the repository currently contains a console-style `main()` application rather than an HTTP server. Therefore `EXPOSE 8080` documents the requested port but the current Java program itself does not create an HTTP server. A real browser-accessible application would need a web server/framework listening on port 8080.

---

## 3.b) Build and run

Build:

```bash
docker build -t library-management:latest .
```

Run:

```bash
docker run -d -p 8080:8080 --name library-management library-management:latest
```

---

## 3.c) Four troubleshooting checks

### 1. Check whether the container is running

```bash
docker ps
```

### 2. Check container logs

```bash
docker logs library-management
```

### 3. Check port mapping

```bash
docker port library-management
```

or:

```bash
docker ps
```

### 4. Inspect the container

```bash
docker inspect library-management
```

Also verify that the application actually listens on port 8080. A plain `java -jar` console program does not automatically become an HTTP server.

---

## 3.d) Container runs but browser gives 404

Possible causes:

1. The Java program is not a web application and does not listen on port 8080.
2. The application is listening on a different port.
3. The requested URL/path does not exist.
4. The application failed internally even though the container remains running.

Check:

```bash
docker logs library-management
```

Check the JAR:

```bash
docker exec -it library-management sh
```

Then:

```bash
java -version
ls -l /app
```

For a real web application, verify the process is listening on 8080 and test from inside the container if suitable.

---

# SET-1 — Maven Web Application / Git / Docker
Repository: `archanareddyse/Lab-Internal-1-LMS.git`

## Part I — Maven

### 1. Basic Maven project

#### a) groupId, artifactId, version

Example:

```xml
<groupId>com.library</groupId>
<artifactId>library-management</artifactId>
<version>1.0-SNAPSHOT</version>
```

#### b) Java source location

```text
src/main/java
```

Test source:

```text
src/test/java
```

#### c) Compile

```bash
mvn compile
```

#### d) Run tests

```bash
mvn test
```

#### e) Generated JAR/WAR location

```text
target/
```

---

## 2. Multi-module Maven project

### a) What is it?

A Maven multi-module project has one parent POM and multiple child modules managed and built together.

Example:

```text
parent
├── library-core
└── library-web
```

### b) Declare modules

In parent `pom.xml`:

```xml
<modules>
    <module>library-core</module>
    <module>library-web</module>
</modules>
```

### c) library-web uses library-core

Declare `library-core` as a dependency in `library-web/pom.xml`:

```xml
<dependency>
    <groupId>com.library</groupId>
    <artifactId>library-core</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### d) Build all modules

```bash
mvn clean install
```

### e) Build order

Maven builds:

```text
library-core → library-web
```

because `library-web` depends on `library-core`.

---

## 3. Maven Compiler Plugin

### 1. Where is the plugin added?

Inside:

```xml
<build>
    <plugins>
        ...
    </plugins>
</build>
```

Example:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.13.0</version>
    <configuration>
        <source>17</source>
        <target>17</target>
    </configuration>
</plugin>
```

### 2. Dependency vs plugin

**Dependency:** library required by the application code.

**Plugin:** tool used by Maven to perform build tasks such as compiling, testing, packaging or generating files.

### 3. Compilation phase

Compilation occurs during:

```text
compile
```

---

## 4. Patch file integration

Apply the patch:

```bash
git apply bug-fix.patch
```

Check conflicts/errors:

```bash
git status
```

Review changes:

```bash
git diff
```

Build:

```bash
mvn clean package
```

Commit:

```bash
git add .
git commit -m "Apply bug fix patch"
```

---

# SET-1 — Git & GitHub

## 1. Initialize and connect using SSH

```bash
git init
git remote add origin git@github.com:archanareddyse/Lab-Internal-1-LMS.git
git branch -M main
git add .
git commit -m "Initial Maven project"
git push -u origin main
```

---

## 2. `.gitignore`

```gitignore
.env
node_modules/
target/
.classpath
.project
.settings/
.idea/
*.iml
```

Then:

```bash
git add .gitignore
git commit -m "Add gitignore"
```

---

## 3. Correct last commit message

```bash
git commit --amend -m "Correct commit message"
```

---

## 4. Compact commit history

```bash
git log --oneline
```

---

## 5. Create and switch to `feature/login`

```bash
git switch -c feature/login
```

---

## 6. Add, commit and push

```bash
git add .
git commit -m "Add login feature"
git push -u origin feature/login
```

---

## 7. Temporarily save work

```bash
git stash
git stash list

git switch main
```

After completing main-branch work:

```bash
git switch feature/login
git stash pop
```

---

## 8. Unstage `app.js`

```bash
git restore --staged app.js
```

The edits remain in the working directory.

---

## 9. Merge conflict

```bash
git switch main
git merge feature/signup
```

Check conflicts:

```bash
git status
```

Edit the conflicted `app.js`, remove:

```text
<<<<<<<
=======
>>>>>>>
```

Stage:

```bash
git add app.js
```

Complete merge:

```bash
git commit -m "Resolve merge conflict in app.js"
```

Push:

```bash
git push origin main
```

---

## 10. Safely undo a pushed commit

```bash
git revert <commit-hash>
git push origin main
```

`git revert` preserves existing history and creates a new reverse commit.

---

# SET-1 — Docker / Tomcat

## 1. Clone

```bash
git clone https://github.com/archanareddyse/Lab-Internal-1-LMS.git
cd Lab-Internal-1-LMS
ls
```

Verify:

```bash
ls pom.xml
```

---

## 2. Dockerfile — Maven build + Tomcat deployment

For a Maven WAR project using the traditional `javax.servlet` APIs, Tomcat 9 is appropriate:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM tomcat:9.0-jdk17-temurin

RUN rm -rf /usr/local/tomcat/webapps/ROOT

COPY --from=build /app/target/*.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```

---

## 3. Build image

```bash
docker build -t library-management:latest .
```

Verify:

```bash
docker images
```

---

## 4. Run Tomcat

Pull:

```bash
docker pull tomcat:9.0-jdk17-temurin
```

Run:

```bash
docker run -d --name library-tomcat -p 7070:8080 library-management:latest
```

Verify:

```bash
docker ps
```

---

## 5. Deploy and verify WAR

If Tomcat is already running separately:

```bash
docker cp target/library-management.war library-tomcat:/usr/local/tomcat/webapps/
```

Check:

```bash
docker exec library-tomcat ls /usr/local/tomcat/webapps/
```

Logs:

```bash
docker logs library-tomcat
```

Browser:

```text
http://localhost:7070
```

or:

```text
http://localhost:7070/library-management
```

depending on the WAR filename.

---

## 6. Port conflict

Check port 7070.

Windows:

```cmd
netstat -ano | findstr :7070
```

Docker:

```bash
docker ps
```

Use another host port:

```bash
docker run -d --name library-tomcat -p 7071:8080 library-management:latest
```

Access:

```text
http://localhost:7071
```

---

## 7. Push Docker image to Docker Hub

Login:

```bash
docker login
```

Tag:

```bash
docker tag library-management:latest <dockerhub-username>/library-management:latest
```

Push:

```bash
docker push <dockerhub-username>/library-management:latest
```

Verify:

```bash
docker images
```

Then check the public repository on Docker Hub.

---

# SET-2 — Online Examination System (OLES)
Repository: `archanareddyse/Lab-Internal-1-OLES.git`

## Part I — Maven

### Q1. Maven Build Failure and Java Compatibility

#### a) Compiler plugin

```text
maven-compiler-plugin
```

#### b) Check Maven Java

```bash
mvn -version
```

#### c) Configure Java 17

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>
```

Or:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.13.0</version>
    <configuration>
        <source>17</source>
        <target>17</target>
    </configuration>
</plugin>
```

#### d) Detailed debugging

```bash
mvn clean package -X
```

---

## Q2. Dependency and JUnit troubleshooting

### a) Dependency tree

```bash
mvn dependency:tree
```

For a particular dependency:

```bash
mvn dependency:tree -Dincludes=groupId:artifactId
```

### b) Check `.m2/repository`

Windows:

```cmd
dir %USERPROFILE%\.m2\repository
```

Linux/macOS:

```bash
ls ~/.m2/repository
```

### c) Different versions of the same dependency

Maven normally uses **dependency mediation**, commonly selecting the dependency version that is nearest to the project in the dependency tree. An explicitly declared direct dependency can be used to control the version.

---

## Q3. Testing and failed tests

### a) Compiled test classes

```text
target/test-classes/
```

### b) JUnit reports

```text
target/surefire-reports/
```

### c) Run a particular test class

```bash
mvn -Dtest=TestClassName test
```

### d) Rerun failed tests

Inspect:

```text
target/surefire-reports/
```

Identify the failed test classes/methods and run them specifically:

```bash
mvn -Dtest=FailedTestClass test
```

For a particular method:

```bash
mvn -Dtest=TestClass#testMethod test
```

---

## Q4. WAR to JAR and executable JAR

### a) Packaging

Change:

```xml
<packaging>war</packaging>
```

to:

```xml
<packaging>jar</packaging>
```

### b) Plugin

Use:

```text
maven-jar-plugin
```

Example:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.4.2</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.example.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

### c) Required information

The fully qualified name of the class containing:

```java
public static void main(String[] args)
```

### d) Build JAR

```bash
mvn clean package
```

---

# SET-2 — Git & GitHub

## 1. Clone MavenWebApp using SSH

```bash
git clone git@github.com:<username>/MavenWebApp.git
cd MavenWebApp
git status
git remote -v
```

## 2. Check status/remotes

```bash
git status
git remote -v
```

## 3. Create `feature/homepage`

```bash
git switch -c feature/homepage
git branch
```

## 4. Add `README.md` and commit

```bash
git add .
git commit -m "Add homepage changes and README"
```

## 5. Retrieve latest remote changes without losing local commit

If working on a feature branch:

```bash
git fetch origin
git rebase origin/main
```

If conflicts occur, resolve them, then:

```bash
git add .
git rebase --continue
```

## 6. Bring feature branch up to date with main

The intended command is:

```bash
git rebase main
```

Or after fetching the remote main:

```bash
git rebase origin/main
```

## 7. Undo an unwanted previous commit without deleting history

```bash
git revert <commit-hash>
```

This creates a new commit reversing that commit.

## 8. Remove a mistakenly committed file but keep it locally

```bash
git rm --cached filename
```

Then:

```bash
git add .gitignore
git commit -m "Stop tracking unwanted file"
```

## 9. Compare current branch with main

Changes:

```bash
git diff main...feature/homepage
```

Commits:

```bash
git log main..feature/homepage --oneline
```

Commits on main but not feature:

```bash
git log feature/homepage..main --oneline
```

## 10. Merge and synchronize

```bash
git switch main
git pull origin main
git merge feature/homepage
git push origin main
git status
```

---

# SET-2 — Docker / Tomcat / Ubuntu

## 1. Clone

```bash
git clone https://github.com/archanareddyse/Lab-Internal-1-OLES.git
cd Lab-Internal-1-OLES
ls pom.xml
```

## 2. Dockerfile

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM tomcat:9.0-jdk17-temurin

RUN rm -rf /usr/local/tomcat/webapps/ROOT

COPY --from=build /app/target/*.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```

## 3. Build, tag and verify

```bash
docker build -t oles:latest .
docker images
```

## 4. Pull and run Tomcat

```bash
docker pull tomcat:9.0-jdk17-temurin
docker run -d --name oles-tomcat -p 7070:8080 oles:latest
docker ps
```

## 5. Deploy WAR

```bash
docker cp target/*.war oles-tomcat:/usr/local/tomcat/webapps/
docker exec oles-tomcat ls /usr/local/tomcat/webapps/
docker logs oles-tomcat
```

Browser:

```text
http://localhost:7070
```

## 6. Ubuntu + Python

```bash
docker pull ubuntu:latest

docker run -dit --name ubuntu-python ubuntu:latest bash

docker exec -it ubuntu-python bash
```

Inside the container:

```bash
apt update
apt install -y python3
python3 --version
```

Create/run a simple program:

```bash
echo 'print("Hello from Python in Docker")' > test.py
python3 test.py
```

## 7. Docker Hub

```bash
docker login
docker tag oles:latest <dockerhub-username>/oles:latest
docker push <dockerhub-username>/oles:latest
```

---

# SET-3 — Maven Web Application / WAR Deployment
Repository: `archanareddyse/Lab-Internal-1-simp-cal.git`

## Part I — Maven

### Q1. Maven Web Application and WAR Deployment

#### a) Packaging

```xml
<packaging>war</packaging>
```

#### b) Standard structure

```text
project/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/
    │   ├── resources/
    │   └── webapp/
    │       ├── WEB-INF/
    │       │   └── web.xml
    │       └── index.jsp
    └── test/
        └── java/
```

#### c) Generate WAR

```bash
mvn clean package
```

#### d) WAR location

```text
target/
```

Example:

```text
target/library-web.war
```

---

# Q2. Servlet API and JSTL

## a) Servlet API

For a Jakarta Servlet 6 application:

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
```

For an older `javax.servlet`/Tomcat 9 application:

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

## b) JSTL

For the traditional `javax` JSTL API:

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

## c) Why `provided`?

The Servlet API is normally supplied by the servlet container, such as Tomcat.

Therefore Maven needs the API during compilation, but it should not package another copy into the application because Tomcat already provides it.

---

# Q3. Maven Build Output and Test Skipping

## a) `mvn clean`

Deletes the previous build output, normally:

```text
target/
```

## b) `mvn install`

Builds the project and installs its artifact into the local Maven repository:

```text
~/.m2/repository
```

## c) Files/directories in `target/`

Examples:

```text
target/classes/
target/test-classes/
target/surefire-reports/
target/*.jar
target/*.war
```

## d) Skip test execution

```bash
mvn clean package -DskipTests
```

If test compilation should also be skipped:

```bash
mvn clean package -Dmaven.test.skip=true
```

---

# Q4. Custom JAR and Patch Integration

## a) Install custom JAR

```bash
mvn install:install-file \
  -Dfile=library-utils.jar \
  -DgroupId=com.library \
  -DartifactId=library-utils \
  -Dversion=1.0 \
  -Dpackaging=jar
```

## b) Declare dependency

```xml
<dependency>
    <groupId>com.library</groupId>
    <artifactId>library-utils</artifactId>
    <version>1.0</version>
</dependency>
```

## c) Verify dependency tree

```bash
mvn dependency:tree
```

---

# SET-3 — Git & GitHub

## 1. Check branches

```bash
git branch
git branch -a
```

The branch marked with `*` is the active branch.

## 2. Create `feature/payment` from latest main

```bash
git switch main
git pull origin main
git switch -c feature/payment
```

## 3. View modifications

```bash
git status
git diff
```

## 4. Stage only one file

```bash
git add file1.java
git status
```

`git status` shows staged and unstaged changes.

## 5. Unstage it

```bash
git restore --staged file1.java
git status
```

The modifications are not lost.

## 6. Stage and commit

```bash
git add .
git commit -m "Implement payment feature"
git show --stat --oneline HEAD
```

## 7. Combine correction with previous commit

Correct the code, then:

```bash
git add .
git commit --amend
```

or:

```bash
git commit --amend -m "Implement payment feature correctly"
```

## 8. Stash changes

```bash
git stash
git stash list

git switch main
# perform required work

git switch feature/payment
git stash pop
```

## 9. Rename branch and push

```bash
git switch feature/payment
git branch -m feature/online-payment

git push -u origin feature/online-payment

git push origin --delete feature/payment
```

The `-u` sets the new branch's upstream tracking branch.

## 10. Tag release

```bash
git tag v1.0
git push origin v1.0
git tag
```

---

# SET-3 — Docker / Ubuntu

## 1. Clone

```bash
git clone https://github.com/archanareddyse/Lab-Internal-1-simp-cal.git
cd Lab-Internal-1-simp-cal
ls pom.xml
```

## 2. Dockerfile

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM tomcat:9.0-jdk17-temurin

RUN rm -rf /usr/local/tomcat/webapps/ROOT

COPY --from=build /app/target/*.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```

If the project uses Jakarta Servlet 6 APIs, use a Tomcat 10.1 runtime instead.

## 3. Build/tag/verify

```bash
docker build -t simp-cal:latest .
docker images
```

## 4. Ubuntu container

```bash
docker pull ubuntu:latest
docker run -dit --name labinternal-1 ubuntu:latest bash
docker ps
```

## 5. Enter Ubuntu and install Git/Nano

```bash
docker exec -it labinternal-1 bash
```

Inside:

```bash
apt update
apt install -y git nano

git --version
nano --version
```

## 6. Docker Hub

```bash
docker login
docker tag simp-cal:latest <dockerhub-username>/simp-cal:latest
docker push <dockerhub-username>/simp-cal:latest
```

---

# CSM-A — SET-1 — AI-OLMS
Repository: `deepthisagar7/AI-OLMS`

## Part I — Maven Web Application Development

### 1.a) Clone and import

```bash
git clone https://github.com/deepthisagar7/AI-OLMS.git
cd AI-OLMS
```

In Eclipse:

**File → Import → Maven → Existing Maven Projects → select project → Finish**

Set the project JDK to Java 17.

---

## 1.b) Corrected WAR + Java 17 POM

The repository's current `pom.xml` contains malformed XML and Java 8 compiler settings. A corrected version is:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.aiolms</groupId>
    <artifactId>ai-olms</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </properties>

    <dependencies>
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <version>6.0.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>AI-OLMS</finalName>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.4.0</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

The repository currently contains `src/main/webapp/index.jsp`, `course.jsp`, and `WEB-INF/web.xml`, so WAR packaging is appropriate.

---

## 1.c) Lifecycle

| Phase | Meaning |
|---|---|
| `clean` | Removes previous build output |
| `compile` | Compiles main Java source |
| `test` | Runs tests |
| `package` | Creates the WAR |
| `install` | Installs the artifact into local `.m2` |

**WAR generation:** `package`

---

## 1.d) Build

```bash
mvn clean package
```

---

## 1.e) Java source/target error

Check:

```bash
java -version
javac -version
mvn -version
```

Make sure `mvn -version` reports Java 17.

Check Windows:

```cmd
echo %JAVA_HOME%
where java
```

Set `JAVA_HOME` to JDK 17, reopen the terminal and verify again.

Then:

```bash
mvn clean package
```

---

## 1.f) WAR builds but Tomcat deployment fails

### Cause 1 — Servlet/Tomcat version mismatch

AI-OLMS uses Jakarta Servlet 6.0.0, which requires a compatible Jakarta-based Tomcat runtime such as Tomcat 10.1.

Check the Tomcat version in logs.

### Cause 2 — Incorrect WAR structure

Check the WAR:

```bash
jar tf target/AI-OLMS.war
```

It should contain web resources and:

```text
WEB-INF/
```

### Cause 3 — Application/deployment configuration problem

Check Tomcat logs:

```bash
docker logs <container-name>
```

or inspect Tomcat's `logs/` directory.

---

## 1.g) Java version mismatch

Check:

```bash
java -version
mvn -version
```

Configure:

```xml
<maven.compiler.source>17</maven.compiler.source>
<maven.compiler.target>17</maven.compiler.target>
```

Same JDK versions should be used because they provide consistent compilation, bytecode compatibility, dependencies and runtime behaviour.

---

# CSM-A SET-1 — Part II Git

## 2.a)

```bash
git init
git status
git add .
git commit -m "Initial commit"

git remote add origin https://github.com/deepthisagar7/AI-OLMS.git

git branch -M main
git push -u origin main
```

## 2.b)

```bash
git commit --amend -m "Added Assignment Module"
```

## 2.c)

```bash
git switch -c feature/course-recommendation
```

## 2.d)

```bash
git restore course.jsp
```

## 2.e) Conflict

```bash
git status
```

Open `course.jsp`, resolve:

```text
<<<<<<< HEAD
=======
>>>>>>> branch
```

Then:

```bash
git add course.jsp
git commit -m "Resolve course.jsp merge conflict"
git push origin feature/course-recommendation
```

If the merge was completed on main:

```bash
git push origin main
```

## 2.f) Staged unwanted files

```bash
git restore --staged .
git status
```

Create `.gitignore`:

```gitignore
target/
.classpath
.project
.settings/
.idea/
*.iml
.env
*.log
```

For already tracked unwanted files:

```bash
git rm --cached filename
```

Then:

```bash
git add .gitignore
git commit -m "Add gitignore"
```

## 2.g)

### Not pushed

```bash
git add .
git commit --amend
```

### Already pushed

```bash
git revert <commit-hash>
git push origin main
```

---

# CSM-A SET-1 — Part III Docker

## 3.a) Multi-stage Dockerfile

Because the repository uses Jakarta Servlet 6.0, use Tomcat 10.1:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM tomcat:10.1-jdk17-temurin

RUN rm -rf /usr/local/tomcat/webapps/ROOT

COPY --from=build /app/target/AI-OLMS.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```

## 3.b)

```bash
docker build -t ai-olms:latest .
docker run -d --name ai-olms -p 7012:8080 ai-olms:latest
```

Browser:

```text
http://localhost:7012
```

## 3.c) Four checks

```bash
docker ps
docker logs ai-olms
docker port ai-olms
docker inspect ai-olms
```

Also verify:

```bash
docker exec -it ai-olms ls /usr/local/tomcat/webapps/
```

## 3.d) 404 troubleshooting

Check whether the WAR exists:

```bash
docker exec -it ai-olms ls -lh /usr/local/tomcat/webapps/
```

Check Tomcat logs:

```bash
docker logs ai-olms
```

Check WAR contents:

```bash
jar tf target/AI-OLMS.war
```

Possible causes:

- WAR was not copied.
- WAR deployment failed.
- Wrong WAR context path was requested.
- Tomcat/Servlet API version is incompatible.
- Application has deployment errors.

If the WAR is named `AI-OLMS.war`, the default context path is usually:

```text
/AI-OLMS
```

unless it is renamed to `ROOT.war`.

---

# Quick Exam Command Sheet

## Maven

```bash
mvn clean
mvn compile
mvn test
mvn package
mvn install
mvn clean package
mvn clean install
mvn dependency:tree
mvn -version
mvn clean package -X
mvn clean package -DskipTests
mvn -Dtest=TestClass test
```

## Git

```bash
git init
git status
git add .
git commit -m "message"
git clone <url>
git remote -v
git branch
git branch -a
git switch -c feature/name
git switch main
git pull
git fetch
git merge branch
git rebase main
git diff
git log --oneline
git show
git restore file
git restore --staged file
git stash
git stash list
git stash pop
git commit --amend
git revert <hash>
git rm --cached file
git tag v1.0
git push origin main
git push -u origin branch
```

## Docker

```bash
docker build -t image:latest .
docker images
docker pull image
docker run -d --name app -p 7070:8080 image:latest
docker ps
docker ps -a
docker logs app
docker port app
docker inspect app
docker exec -it app bash
docker cp file app:/path/
docker tag image:latest username/image:latest
docker login
docker push username/image:latest
```

---

# Repository-specific findings

## `deepthisagar7/library-management`

The repository currently has:

- `pom.xml`
- `src/main/java/com/library/LibraryManagement.java`
- a `target/` directory already committed.

The current `pom.xml` has malformed closing tags and Java compiler values that are not Java 17. The Java class contains a valid `main()` method, so the appropriate exam solution is a Java 17 executable JAR with `maven-jar-plugin`.

## `deepthisagar7/AI-OLMS`

The repository currently contains:

```text
src/main/webapp/index.jsp
src/main/webapp/course.jsp
src/main/webapp/WEB-INF/web.xml
```

Its `pom.xml` is malformed and currently uses Java 8 settings. It uses `jakarta.servlet-api:6.0.0`, so a Jakarta-compatible Tomcat 10.1 runtime is the appropriate Docker runtime.

## GitHub write limitation

The connected GitHub account can read these public repositories but currently reports **pull-only permission** for them. Therefore I could not directly commit/push the `pom.xml` correction to GitHub from this session. The corrected `pom.xml` is included above so you can replace the repository version locally and run:

```bash
mvn clean package
git add pom.xml
git commit -m "Fix Maven configuration for Java 17"
git push origin main
```
