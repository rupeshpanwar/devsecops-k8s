pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 

            }
        }   //stage ending Build Artifact

      stage('Build Artifact') {
            steps {
              sh "mvn test"

            }
        }   //stage ending Unit test
    }
}