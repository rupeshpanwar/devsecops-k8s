pipeline {
  agent {
          node {
              label "docker"
              customWorkspace "/tmp/deployment"
            }
        }

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "rupeshpanwar/numeric-app:${GIT_COMMIT}"
    applicationURL = "http://142.93.213.194"
    applicationURI = "compare/51"
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
      }   //stage ending Unit test

      stage('Mutation Tests - PIT') {
          steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
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

      // stage('Vulnerability Scan') {
      //     steps {
      //       sh "mvn dependency-check:check"
      //     }
         
      // } // stage ending Vulnerability Scan

      stage('Vulnerability Scan') {
          steps {
            parallel(
              "Dependency Scan": {
                sh "mvn dependency-check:check"
              },
              "Trivy Scan": {
                sh "bash trivy-docker-image-scan.sh"
              },
            "OPA Conftest": {
            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
             }
            )
          }
    } // stage ending Vulnerability Scan

      stage('Docker build & push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                  sh 'printenv'
                  sh 'sudo docker build -t rupeshpanwar/numeric-app:""$GIT_COMMIT"" .'
                  sh 'docker push rupeshpanwar/numeric-app:""$GIT_COMMIT""'
                }
            }
      }   //stage ending Docker build and push 

    //   stage('Vulnerability Scan - K8s') {
    //   steps {
    //     sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
    //   }
    // } // stage ending  Vulnerability Scan - K8s

    stage('Vulnerability Scan - Kubernetes') {
      steps {
        parallel(
          "OPA Scan": {
            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
          },
          "Kubesec Scan": {
            sh "bash kubesec-scan.sh"
          },
          "Trivy Scan": {
            sh "bash trivy-k8s-scan.sh"
          }
        )
      }
    } // stage ending  Vulnerability Scan - K8s

      // stage('Kubernetes Deployment - DEV') {
      //   steps {
      //       withKubeConfig([credentialsId: 'kubeconfig']) {
      //         sh "sed -i 's#replace#rupeshpanwar/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
      //         sh "kubectl apply -f k8s_deployment_service.yaml"
      //       }
      //   }
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
    }  // stage ending k8s deployment -  DEV

      stage('Integration Tests - DEV') {
            steps {
                    script {
                      try {
                        echo "successfully started"
                        // withKubeConfig([credentialsId: 'kubeconfig']) {
                        //   sh "bash integration-test.sh"
                        }
                      } catch (e) {
                        withKubeConfig([credentialsId: 'kubeconfig']) {
                          sh "kubectl -n default rollout undo deploy ${deploymentName}"
                        }
                        throw e
                      }
                    }
                 }
            } //stage ending Integration testing 

        stage('OWASP ZAP - DAST') {
          steps {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh 'bash zap.sh'
            }
          }
    } // stage ending owasp zap - dast

    } // Stages section end here

   

    post {
          always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report'])
          }
     }
}