pipeline {
    agent { label 'ec2-build-agent' } // Tells Jenkins to run this ON the EC2

    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/YOUR_USERNAME/Jenkins-Zero-To-Hero.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                // We use the Maven wrapper already in the repo
                // The scan sends data to localhost:9000 (which is on the EC2)
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                    sh "./mvnw sonar:sonar -Dsonar.host.url=http://localhost:9000"
                }
            }
        }

        stage('Build & Package') {
            steps {
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                    sh "./mvnw clean package -DskipTests"
                }
            }
        }

        stage('Docker Deploy') {
            steps {
                dir('java-maven-sonar-argocd-helm-k8s/spring-boot-app') {
                    sh """
                    docker build -t my-tax-app:latest .
                    docker stop tax-app-container || true
                    docker rm tax-app-container || true
                    docker run -d --name tax-app-container -p 8081:8080 my-tax-app:latest
                    """
                }
            }
        }
    }
}
