pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
      stage('Unit test') {
            steps {
              sh "mvn test"
            }
        } 
        stage('Mutation Tests - PIT') {
              steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
              }
          } 
        stage('Vulnerability Scan - Docker') {
              steps {
                parallel(
                  "Dependency Scan": {
                    sh "mvn dependency-check:check"
                  },
                  "Trivy Scan": {
                    sh "bash trivy-docker-image-scan.sh"
                  },
                  "OPA Conftest":{
                    sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
                  }   
                )
              }
            }

        stage('SonarQube - SAST') {
          steps {
            withSonarQubeEnv('sonarqube') {
              sh "mvn sonar:sonar \
                      -Dsonar.projectKey=numeric-application \
                      -Dsonar.host.url=http://10.0.2.15:9000"
            }
            timeout(time: 2, unit: 'MINUTES') {
              script {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        }
        stage('Docker build and push') {
            steps {
              withDockerRegistry([credentialsId: "dockerhub", url: ""]){
              sh 'printenv'
              sh 'docker build -t rishabh1234/numeric-app:""$GIT_COMMIT"" .'
              sh 'docker push rishabh1234/numeric-app:""$GIT_COMMIT""'
              }
            }
        }
        stage('Vulnerability Scan - Kubernetes') {
          steps {
                sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
          }
        }
        // stage('Kubernetes Dev Deployment') {
        //     steps {
        //         withKubeConfig([credentialsId: 'kubeconfig']) {
        //         sh "sed -i 's#replace#rishabh1234/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
        //         sh "kubectl apply -f k8s_deployment_service.yaml"
        //         }
        //     }
        // }
          stage('K8S Deployment - DEV') {
            steps {
              parallel(
                "Deployment": {
                  withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh "bash k8s-deployment.sh"
                  }
                },
                "Rollout Status": {
                  withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh "bash k8s-deployment-rollout-status.sh"
                  }
                }
              )
            }
          }
    }
    post {
      always {
        junit 'target/surefire-reports/*.xml'
        jacoco execPattern: 'target/jacoco.exec'
        pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
    }
}
