pipeline {
    agent any

    environment {
        REGISTRY          = "docker.io"
        REGISTRY_NAMESPACE= "ramm978"               // your Docker Hub username
        IMAGE_NAME        = "sampleapp"             // your app image name
        K8S_NAMESPACE     = "demo"                  // Kubernetes namespace
        SONARQUBE_SERVER  = "SonarQube"             // Jenkins SonarQube server name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'name.developer',
                    credentialsId: 'github-token',
                    url: 'https://github.com/1mohanr/SampleApp.git'
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    echo "✅ Checked out code. Image tag will be: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Java App') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=sampleapp \
                          -Dsonar.host.url=http://3.107.198.86:9000 \
                          -Dsonar.login=${SONARQUBE_TOKEN}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t $REGISTRY/$REGISTRY_NAMESPACE/$IMAGE_NAME:$IMAGE_TAG .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred-id',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $REGISTRY/$REGISTRY_NAMESPACE/$IMAGE_NAME:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kube-config']) {
                    sh """
                        kubectl set image deployment/sampleapp-deployment \
                          sampleapp=$REGISTRY/$REGISTRY_NAMESPACE/$IMAGE_NAME:$IMAGE_TAG \
                          -n $K8S_NAMESPACE
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withKubeConfig([credentialsId: 'kube-config']) {
                    sh """
                        kubectl get pods -n $K8S_NAMESPACE
                        kubectl get svc -n $K8S_NAMESPACE
                    """
                }
            }
        }
    }
}

