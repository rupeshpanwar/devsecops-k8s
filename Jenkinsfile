pipeline {
  agent {
          node {
              label "docker"
              customWorkspace "/tmp/deployment"
            }
        }

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

      stage('Mutation Tests - PIT') {
          steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
              always {
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              }
          }
      }   // stage ending PIT mutations test

      stage('SonarQube - SAST') {
          steps {
                withSonarQubeEnv('SonarQube') {
                     sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://142.93.213.194:9000"
                }
        //         timeout(time: 2, unit: 'MINUTES') {
        //   script {
        //     waitForQualityGate abortPipeline: false
        //   }
        // }
          }
      } // stage ending SonarQube - SAST

      stage('Vulnerability Scan') {
          steps {
            sh "mvn dependency-check:check"
          }
          post {
            always {
              dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            }
          }
      } // stage ending Vulnerability Scan

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