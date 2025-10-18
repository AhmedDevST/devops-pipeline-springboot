pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
 triggers {
        pollSCM('* * * * *')
    }
    stages {
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
