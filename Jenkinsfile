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
                // This 'withSonarQubeEnv' block pulls the URL/Token from Jenkins settings
                withSonarQubeEnv('SonarQube-Server') { 
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
