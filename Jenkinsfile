pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://3.107.189.219:9000'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/name.developer']],
                          userRemoteConfigs: [[url: 'https://github.com/1mohanr/Mohan-End-to-End-Assignment.git']]])
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'Sample_app', variable: 'SONAR_TOKEN')]) {
                    echo 'Running SonarQube analysis...'
                    sh """
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=SampleApp \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Build & Package') {
            steps {
                echo 'Building and packaging the project...'
                sh 'mvn clean package'
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded. Archiving artifacts and publishing test results...'
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            junit 'target/surefire-reports/*.xml'
        }

        failure {
            echo 'Pipeline failed. Check logs for details.'
        }

        always {
            echo 'Pipeline finished.'
        }
    }
}

