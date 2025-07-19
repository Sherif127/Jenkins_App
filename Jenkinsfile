pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'leoughhh/jenkins_app'
        IMAGE_TAG = "v${BUILD_ID}"
        DEPLOYMENT_FILE = 'deployment.yaml'
    }

    stages {
        stage('Test App') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build App') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                        docker login -u ${USER} -p ${PASS}
                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Clean Local Docker Image') {
            steps {
                sh "docker rmi ${DOCKER_IMAGE}:${IMAGE_TAG}"
            }
        }

        stage('Update deployment.yaml') {
            steps {
                sh """
                    sed -i 's|image:.*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|' ${DEPLOYMENT_FILE}
                    cat ${DEPLOYMENT_FILE}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f ${DEPLOYMENT_FILE}"
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished!'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
