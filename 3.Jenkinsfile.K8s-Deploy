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
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
      }
      }   //stage ending Unit test

      stage('Docker build & push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                  sh 'printenv'
                  sh 'docker build -t rupeshpanwar/numeric-app:""$GIT_COMMIT"" .'
                  sh 'docker push rupeshpanwar/numeric-app:""$GIT_COMMIT""'
                }
            }
      }   //stage ending Docker build and push 

      stage('Kubernetes Deployment - DEV') {
        steps {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "sed -i 's#replace#rupeshpanwar/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh "kubectl apply -f k8s_deployment_service.yaml"
            }
        }
      } // stage ending k8s deployment -  DEV

    }
}