pipeline {
    agent { label 'docker-agent' }

    environment {
        APP_NAME = "java-app"
        DOCKER_IMAGE = "sriven0309/java-app"
        SONAR_HOST_URL = "http://localhost:9000"
        SONAR_PROJECT_KEY = "java-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Java Application') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Run Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Analyze Code with SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    bat """
                        mvn sonar:sonar ^
                          -Dsonar.projectKey=%SONAR_PROJECT_KEY% ^
                          -Dsonar.host.url=%SONAR_HOST_URL% ^
                          -Dsonar.login=%SONAR_TOKEN%
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                bat 'docker run --rm aquasec/trivy:0.69.3 image %DOCKER_IMAGE%:latest'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                    bat 'docker push %DOCKER_IMAGE%:latest'
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                bat 'helm upgrade --install java-app .\\java-app-chart --kubeconfig %USERPROFILE%\\.kube\\config'
                bat 'kubectl --kubeconfig %USERPROFILE%\\.kube\\config rollout status deployment/java-app'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}