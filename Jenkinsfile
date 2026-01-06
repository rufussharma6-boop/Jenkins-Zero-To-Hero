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
                        // We use 'mvn' instead of './mvnw' because the wrapper is missing
                        sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar"
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 60, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                    sh """
                    # Build the .jar file first
                    mvn clean package -DskipTests
                    
                    # Build and run the container
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
