#### Jenkinsfile Contains the Deployment of App on Docker Container running on Port 8006 which is itself in Jenkins Server.
#### To run the Container on Desire Port Please change the Port in Dockerfile.
#### But our ultimate goal is Deployment of App on K8s Cluster which is under Process

pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
        DOCKER_IMAGE_NAME = 'prafulitankar/psi:boardgame'
        CONTAINER_NAME = 'webapps-01'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prafulitankar/Boardgame.git'
            }
        }  
        
        stage('compilation') {
            steps {
                sh "mvn compile"
            }
        }  
        
        stage('test case') {
            steps {
                sh "mvn test"
            }
        } 
        
        stage('File system Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        } 
        
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred' 
                }
            }
        }
        
        stage('Publish Artifact') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
             }
           }
        }
        
        stage('Build and tag docker image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred' , toolName: 'docker') {
                            sh "docker build -t prafulitankar/psi:boardgame ."
                }
               }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html prafulitankar/psi:boardgame "
            }
        }
        
        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push prafulitankar/psi:boardgame"
                    }
               }
            }
        }

        stage('Build Docker Container') {
            steps {
                script {
                    // Run Docker container from image
                    docker.image("${DOCKER_IMAGE_NAME}").run('-d -p 8006:8080 --name ${CONTAINER_NAME}') // Example run options: detached mode, port mapping
                }
            }
        }
        
        /*stage('K8s Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.36.71:6443') {
                         sh "kubectl apply -f deployment-service.yaml"
                    
                }
               
            }
        }
        
        stage('verify Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.36.71:6443') {
                         sh "kubectl get pods -n webapps"
                         sh "kubectl get svc -n webapps"
                }
            }
        } */
    }
    
    post {
    success {
      script {
          def jobName = env.JOB_NAME
          def buildNumber = env.BUILD_NUMBER
          def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
          def bannerColor = pipelineStatus.toUpperCase() == 'BUILD IS SUCCESS' ? 'green' : 'red'

          def body = """
              <html>
              <body>
              <div style="border: 4px solid ${bannerColor}; padding: 10px;">
              <h2>${jobName} - Build ${buildNumber}</h2>
              <div style="background-color: ${bannerColor}; padding: 10px;">
              <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
              </div>
              <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
              </div>
              </body>
              </html>
          """

          emailext (
              subject: "Build Success : ${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
              body: body,
              to: 'praful.itankar@gmail.com',
              from: 'jenkins@example.com',
              replyTo: 'jenkins@example.com',
              mimeType: 'text/html',
              attachmentsPattern: 'trivy-image-report.html'
          )
      }
  }

  failure {
      script {
          def jobName = env.JOB_NAME
          def buildNumber = env.BUILD_NUMBER
          def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
          def bannerColor = pipelineStatus.toUpperCase() == 'BUILD is Failed' ? 'green' : 'red'

          def body = """
              <html>
              <body>
              <div style="border: 4px solid ${bannerColor}; padding: 10px;">
              <h2>${jobName} - Build ${buildNumber}</h2>
              <div style="background-color: ${bannerColor}; padding: 10px;">
              <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
              </div>
              <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
              </div>
              </body>
              </html>
          """

          emailext (
              subject: "Build Failed : ${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
              body: body,
              to: 'praful.itankar@gmail.com',
              from: 'jenkins@example.com',
              replyTo: 'jenkins@example.com',
              mimeType: 'text/html',
              attachmentsPattern: 'trivy-image-report.html'
          )
      }
  }
}
}


