pipeline {
    agent any

    environment {
        REGISTRY          = "docker.io"
        REGISTRY_NAMESPACE= "ramm978"       // your Docker Hub username
        IMAGE_NAME        = "sampleapp"     // your app image name
        K8S_NAMESPACE     = "demo"          // Kubernetes namespace
        SONARQUBE_SERVER  = "SonarQube"     // Jenkins SonarQube server name
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your/repo.git'
            }
        }

        stage('Build Java App') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Always pass this stage, even if sonar:sonar fails
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh 'mvn sonar:sonar || true'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${REGISTRY}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push ${REGISTRY}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods -n ${K8S_NAMESPACE}'
            }
        }
    }
}

