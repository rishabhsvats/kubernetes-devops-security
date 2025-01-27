# kubernetes-devops-security

- To run Sonarqube service use following docker run command
~~~
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
~~~
