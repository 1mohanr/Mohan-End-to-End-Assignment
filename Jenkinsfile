pipeline {
    agent any

    environment {
        REGISTRY           = "docker.io"
        REGISTRY_NAMESPACE = "ramm978"        // your Docker Hub username
        IMAGE_NAME         = "sampleapp"      // your app image name
        K8S_NAMESPACE      = "demo"           // Kubernetes namespace
        SONARQUBE_SERVER   = "SonarQube"      // SonarQube server configured in Jenkins
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "${env.BRANCH_NAME}"]],
                    userRemoteConfigs: [[
                        url: 'https://github.com/1mohanr/mohan-end-to-end-assignment.git',
                        credentialsId: 'github-token'
                    ]]
                ])
                script {
                    env.IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "✅ Checked out code. Image tag will be: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Java App') {
            steps {
                script {
                    try {
                        sh "mvn clean package -DskipTests"
                    } catch (Exception e) {
                        error "❌ Maven build failed. Stopping pipeline."
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh "mvn sonar:sonar"
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sh "docker build -t ${REGISTRY}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${REGISTRY}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sh "kubectl set image deployment/${IMAGE_NAME} ${IMAGE_NAME}=${REGISTRY}/${REGISTRY_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG} -n ${K8S_NAMESPACE}"
            }
        }

        stage('Verify Deployment') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sh "kubectl get pods -n ${K8S_NAMESPACE}"
                sh "kubectl get svc -n ${K8S_NAMESPACE}"
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

