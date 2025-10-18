pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    triggers {
        // Poll GitHub every 5 minutes
        pollSCM('H/5 * * * *')
    }
     environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('checkout') {
            steps {
                git credentialsId: 'jen_dock_git', url: 'https://github.com/AhmedDevST/devops-pipeline-springboot'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
       stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                            sh '''
                $SCANNER_HOME/bin/sonar-scanner \
                  -Dsonar.projectName=demo_sonar \
                  -Dsonar.projectKey=demo_sonar \
                  -Dsonar.java.binaries=.
                '''
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps { 
                script {
                    withDockerRegistry(credentialsId: 'dockerHub_cred', toolName: 'docker') {
                        sh "docker build -t demoapp:latest -f Dockerfile ."
                        sh "docker tag demoapp:latest ahmed0987/demoapp:latest"
                        sh "docker push ahmed0987/demoapp:latest"
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                script {
                     withDockerRegistry(credentialsId: 'dockerHub_cred', toolName: 'docker') {
                        sh "docker stop demoapp || true"
                        sh "docker rm demoapp || true"
                        sh "docker run -d --name demoapp -p 9090:9090 ahmed0987/demoapp:latest"
                    }   
                }
            }
        }
    }
}
