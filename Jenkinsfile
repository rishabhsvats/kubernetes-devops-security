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
        stage('Docker build and push') {
            steps {
              withDockerRegistry([credentialsId: "dockerhub", url: ""])
              sh 'printenv'
              sh 'docker build -t rishabh1234/numeric-app:""$GIT_COMMIT"" .'
              sh 'docker push rishabh1234/numeric-app:""$GIT_COMMIT""'
              }
            }
    }
}
