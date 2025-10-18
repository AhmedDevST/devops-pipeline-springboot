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
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t demo-app .'
            }
        }
    }
}
