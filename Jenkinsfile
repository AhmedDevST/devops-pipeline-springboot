pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
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
        
        stage('Build & Push Docker Image') {
            steps { 
                script{
                    withDockerRegistry(credentialsId: 'dockerHub_cred', toolName: 'docker') {
                        sh "docker build -t demoApp: latest -f docker/Dockerfile "
                        sh "docker tag shopping: latest ahmed0987/demoApp:latest" 
                        sh "docker push ahmed0987/shopping:latest"
                      }
                }
            }
        }
    }
}
