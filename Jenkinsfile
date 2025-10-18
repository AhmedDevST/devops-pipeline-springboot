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
                script {
                    withDockerRegistry(credentialsId: 'dockerHub_cred', toolName: 'docker') {
                        sh "docker build -t demoapp:latest -f Dockerfile ."
                        sh "docker tag demoapp:latest ahmed0987/demoapp:latest"
                        sh "docker push ahmed0987/demoapp:latest"
                    }
                }
            }
        }
    }
}
