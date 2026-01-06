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
                // Ensure 'sonar-test' matches exactly in Jenkins -> System -> SonarQube
                withSonarQubeEnv('sonar-test') { 
                    dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                        sh "chmod +x mvnw"
                        sh "./mvnw sonar:sonar"
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                // This waits for SonarQube to finish processing before continuing
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                    sh """
                    # Build the image inside the app directory
                    docker build -t my-app:latest .
                    
                    # Remove existing container if it exists to avoid port/name conflicts
                    docker stop my-app-container || true
                    docker rm my-app-container || true
                    
                    # Run the new container
                    docker run -d --name my-app-container -p 80:8080 my-app:latest
                    """
                }
            }
        }
    }
}
