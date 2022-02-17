pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 

            }
      }   //stage ending Build Artifact

      stage('Unit test') {
            steps {
              sh "mvn test"
            }
            post {
              always {
                junit 'taget/surface-reports/*.xml'
                jococo execPattern 'target/jococo.exec'
              }
            }
      }   //stage ending Unit test
    }
}