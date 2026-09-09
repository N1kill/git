KESHAV MEMORIAL INSTITUTE OF TECHNOLOGY firs tdo part 2

                                          (AN AUTONOMOUS INSTITUTE)
        Accredited by NBA & NAAC, Approved by AICTE, Affiliated to JNTUH, Hyderabad	
                                       A.Y 2026-2027
             Department of Computer Science & Engineering (AI/ML)

Lab Internal I
Subject Name: Software Engineering                                      Subject Code: 23CC501PC
Year and Semester:   III / I                                                   Branch /Section: CSM-A Faculty: Y Deepthi                                                            Lab Internal: 07/09/2026 
SET–3
Library Management system
Max. Marks: 100
________________________________________

Part I – Maven Java Application Development [40 Marks]
You are working on the Library Management System available at:
https://github.com/deepthisagar7/library-management.git
The project has been imported into Eclipse/IntelliJ as a Maven Java project. The existing pom.xml contains configuration errors, and the project is currently configured for Java 8. The application must be successfully built as a Maven Java application using Java 17, generating a JAR file.

1.a)Write the Git command to clone the Library Management System repository. Also mention the steps to import the cloned project as a Maven project in Eclipse/IntelliJ. [4 Marks]
1.b)The existing pom.xml contains errors in project configuration, packaging type, and Java compiler settings.
Identify and correct the configuration so that the Library Management System can be successfully built as a Maven Java application using Java 17 and generates a JAR file.
Write the relevant corrected portion of pom.xml. [10 Marks]
1.c)Explain the following Maven lifecycle phases:
•	clean
•	compile
•	test
•	package
•	install
Which phase generates the JAR file? [5 Marks]
1.d)Write the Maven command to clean and package the Library Management System. [3 Marks]
1.e)After changing the project to Java 17, a developer executes:
mvn clean package
but receives an error related to an unsupported Java source or target version.
Explain how you would troubleshoot and resolve the problem.
Include the commands you would use to check:
•	Installed Java version
•	Maven's Java version [6 Marks]
1.f)The Maven build succeeds and produces:
target/library-management.jar
However, the JAR file fails to execute.
Identify three possible causes and explain how you would troubleshoot each problem. [6 Marks]
1.g)A developer reports:
"The project builds successfully on my machine but fails on another developer's machine."
The first machine uses Java 17, while the second machine uses Java 8.
As a team member, explain:
1.	How you would identify the version mismatch.
2.	How Maven can be configured to require Java 17.
3.	Why the same JDK version should be used across development and deployment environments. [6 Marks]



Part II – Git & GitHub [40 Marks]
You are developing the Library Management System. The project must be maintained using Git and synchronized with the given GitHub repository.
GitHub Repository:
https://github.com/deepthisagar7/library-management.git
	
2.a)The project has not yet been initialized as a Git repository.
Write the Git commands to:
1.	Initialize Git.
2.	Check repository status.
3.	Add all project files.
4.	Create an initial commit.
5.	Connect the local repository to the given GitHub repository.
6.	Push the project to the main branch. [6 Marks]

2.b)You accidentally entered the commit message:
Added Book Managment Module
instead of:
Added Book Management Module
The commit has not yet been pushed to GitHub.
Write the Git command to correct the commit message. [4 Marks]
2.c)Create a new branch named:
feature/book-management
and switch to that branch.
Write the required command. [4 Marks]
2.d)A developer accidentally deleted Book.java before committing the deletion.
Write the Git command to restore the deleted file. [4 Marks]
2.e)You are working on the feature/book-management branch. Another developer has modified the same section of a book-related source file on the main branch.
When you merge the branches, Git reports a merge conflict.
Explain the complete process to:
1.	Identify the conflicted file.
2.	Open and resolve the conflict.
3.	Stage the resolved file.
4.	Complete the merge.
5.	Push the changes to GitHub.
Include the relevant Git commands. [8 Marks]
2.f)A developer accidentally executes:
git add .
Several unwanted IDE configuration files, compiled files, and temporary files are now staged.
Explain how to:
1.	Remove unwanted files from the staging area without deleting them.
2.	Check which files are staged.
3.	Create a suitable .gitignore file.
4.	Prevent such files from being tracked in future commits. [6 Marks]
2.g)A feature has been completed and committed, but the developer realizes that the latest commit contains an incorrect implementation.
Explain two appropriate ways to handle this situation:
•	When the commit has not been pushed
•	When the commit has already been pushed
Mention suitable Git commands and explain when each approach should be used. [8 Marks]



Part III – Dockerization of Maven Java Application [20 Marks]
The Library Management System Maven Java application has been successfully built as a JAR file and is ready for containerization using Docker.
The application uses Java 17 and Maven.
The generated JAR file is:
target/library-management.jar
The application should be accessible through:
http://localhost:8080
3.a)Write a Dockerfile that:
1.	Uses a suitable Java 17 base image.
2.	Copies the generated JAR file into the Docker image.
3.	Exposes port 8080.
4.	Specifies the command to execute the JAR file.
5.	Creates a suitable container for running the Library Management System. [8 Marks]
3.b)Write the Docker commands to:
1.	Build an image named library-management:latest.
2.	Run the container in detached mode.
3.	Map host port 8080 to container port 8080. [4 Marks]
3.c)You run:
docker run -d -p 8080:8080 library-management:latest
but cannot access the application at:
http://localhost:8080
Write four troubleshooting checks/commands you would perform to identify the problem. [4 Marks]
3.d)The Docker container is running, but the browser displays 404 Not Found.
The Dockerfile contains:
COPY target/library-management.jar app.jar
Identify possible reasons for the 404 error and explain how you would verify whether the Java application is running correctly inside the container. [4 Marks]
