# kubernetes-devops-security

- To run Sonarqube service use following docker run command
~~~
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000  -v /path/sonarqube/data:/opt/sonarqube/data -v /path/sonarqube/extensions:/opt/sonarqube/extensions sonarqube:latest
~~~


- NodeJS Microservice - Docker Image

~~~
docker run -p 8787:5000 siddharth67/node-service:v1

curl localhost:8787/plusone/99
~~~


- NodeJS Microservice - Kubernetes Deployment

~~~
$ kubectl create deploy node-app --image siddharth67/node-service:v1

$ kubectl expose deploy node-app --name node-service --port 5000 --type ClusterIP

$ curl node-service-ip:5000/plusone/99
~~~

