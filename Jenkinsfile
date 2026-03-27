pipeline {
    agent any

    environment {
        APP_NAME = "java-app"
        DOCKER_IMAGE = "sriven0309/java-app"
        SONAR_HOST_URL = "http://sonarqube:9000"
        SONAR_PROJECT_KEY = "java-app"
        KUBECONFIG = "/var/jenkins_home/.kube/config"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Java Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Analyze Code with SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                sh '''
                    docker run --rm aquasec/trivy:0.69.3 image ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    helm upgrade --install java-app ./java-app-chart --kubeconfig /var/jenkins_home/.kube/config
                    kubectl --kubeconfig /var/jenkins_home/.kube/config rollout status deployment/java-app
                '''
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