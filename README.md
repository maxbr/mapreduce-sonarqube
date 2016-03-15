Description
-----------

Simple implementation of Word-Count example. 

Input is read from directory */tmp/wordcount/in*, and output is written to */tmp/wordcount/out*.

Steps
---
1. git clone https://bitbucket.org/maxbr/mapreduce-sonarqube.git
2. cd mapreduce-sonarqube
3. vagrant up
4. vagrant ssh 
5. cd Hadoop-Word-Count
6. mkdir -p /tmp/wordcount/in
7. cp ./README.md /tmp/wordcount/in/
8. mvn clean package
9. java -javaagent:${HOME}/.m2/repository/org/jacoco/org.jacoco.agent/0.7.5.201505241946/org.jacoco.agent-0.7.5.201505241946-runtime.jar=destfile=${PWD}/target/jacoco-it.exec -cp \`hadoop classpath\`:target/wordcount-0.0.1-SNAPSHOT.jar com.igalia.wordcount.App
10. mvn sonar:sonar -Dsonar.username=admin -Dsonar.password=admin -Dsonar.host.url=http://127.0.0.1:9000/ -Dsonar.projectName="Hadoop WordCount" -Dsonar.projectKey=wordcount:sonar:key
11. rm -rf /tmp/wordcount/out

