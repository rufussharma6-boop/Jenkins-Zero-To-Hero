pipeline {
    agent { label 'ec2-worker' } 

    stages {
        stage('Checkout') {
            steps {
                // Jenkins uses the GitHub plugin to pull the latest code
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-test') { 
                    dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                        sh "chmod +x mvnw" // Ensure the wrapper has execution permission
                        sh "./mvnw sonar:sonar"
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                // Optional: This stops the pipeline if SonarQube finds too many bugs
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Build & Deploy') {
            steps {
                sh "docker build -t my-app . && docker run -d -p 80:80 my-app"
            }
        }
    }
}
