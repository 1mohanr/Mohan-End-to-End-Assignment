pipeline {
    agent any

    environment {
        REGISTRY           = "docker.io"
        REGISTRY_NAMESPACE = "ramm978"
        IMAGE_NAME         = "sampleapp"
        K8S_NAMESPACE      = "demo"
        SONARQUBE_SERVER   = "SonarQubeServer"
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    // sanitize branch name for Docker tag
                    env.BRANCH_NAME_SAFE = env.BRANCH_NAME.replaceAll('/', '-')
                }
            }
        }

        stage('Build Java App') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${REGISTRY_NAMESPACE}/${IMAGE_NAME}:${env.BRANCH_NAME_SAFE}-${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub', url: "https://${REGISTRY}"]) {
                    sh "docker push ${REGISTRY_NAMESPACE}/${IMAGE_NAME}:${env.BRANCH_NAME_SAFE}-${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/${IMAGE_NAME} \
                ${IMAGE_NAME}=${REGISTRY_NAMESPACE}/${IMAGE_NAME}:${env.BRANCH_NAME_SAFE}-${BUILD_NUMBER} \
                -n ${K8S_NAMESPACE}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "kubectl rollout status deployment/${IMAGE_NAME} -n ${K8S_NAMESPACE}"
                sh "kubectl get pods -n ${K8S_NAMESPACE}"
            }
        }
    }

    post {
        failure {
            echo "⚠️ Pipeline failed. Check logs for details."
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
    }
}

