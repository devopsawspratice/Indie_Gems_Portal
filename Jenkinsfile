pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/Game"
        IMAGE_NAME = "indie-gems"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "indie-gems-container"
        PORT = "9676"
        DOCKERHUB_USER = "devopsawspratice"
        DOCKER_CREDS = "docker"
        CONTAINER_PORT = "80"
        AWS_REGION = "us-east-1"
        EKS_CLUSTER = "saran"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        NOTIFY_EMAIL = "satyanarayana.gidituri666@gmail.com"
    }

    stages {

        stage('Checkout Code') {
            steps {
                dir("${WORK_DIR}") {
                    git branch: 'main', url: 'https://github.com/devopsawspratice/Indie_Gems_Portal.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir("${WORK_DIR}") {
                    sh '''
                        docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
                        docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh '''
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Update K8s Image') {
            steps {
                sh '''
                    sed -i "s|image:.*|image: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yml
                '''
            }
        }

        stage('Configure EKS Access') {
            steps {
                sh '''
                    export PATH=$PATH:/usr/local/bin
                    aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                    kubectl config current-context
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yml
                    kubectl apply -f k8s/service.yml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment python-devops-app || true
                    kubectl get pods -o wide
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            sendMail("SUCCESS")
        }
        failure {
            sendMail("FAILURE")
        }
    }
}

def sendMail(status) {
    emailext(
        subject: "${status}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
            <h2>Pipeline Status: ${status}</h2>
            <p><b>Job:</b> ${env.JOB_NAME}</p>
            <p><b>Build:</b> ${env.BUILD_NUMBER}</p>
            <p><b>URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
        """,
        to: "${env.NOTIFY_EMAIL}",
        mimeType: 'text/html'
    )
}
