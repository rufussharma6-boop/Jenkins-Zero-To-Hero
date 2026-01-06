pipeline {
    agent { label 'ec2-worker' } 

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-test') { 
                    dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                        sh "ls -la" // This will show us if mvnw is actually here
                        sh "chmod +x mvnw"
                        sh "./mvnw sonar:sonar"
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                    sh """
                    docker build -t my-app:latest .
                    docker stop my-app-container || true
                    docker rm my-app-container || true
                    docker run -d --name my-app-container -p 80:8080 my-app:latest
                    """
                }
            }
        }
    }
}
