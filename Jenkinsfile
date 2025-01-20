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
            post{
              always{
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exe'
              }
            }
        } 
        stage('Mutation Tests - PIT') {
              steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
              }
              post {
              always {
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              }
            }
          } 
      stage('SonarQube - SAST') {
            steps {
              sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://localhost:9000 -Dsonar.token=sqp_2c437cf7cff69fb4446b43b9f7054100c409d3f9"
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
        stage('Kubernetes Dev Deployment') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "sed -i 's#replace#rishabh1234/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
                }
            }
        }
    }
}
