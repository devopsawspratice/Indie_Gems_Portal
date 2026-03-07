pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/Game"
        IMAGE_NAME = "indie-gems"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        CONTAINER_NAME = "indie-gems-container"
        PORT = "9676"   // External port for app
        DOCKERHUB_USER  = "9397054542"
        DOCKER_CREDS    = "DockerhubCred"
        CONTAINER_PORT  = "80"
    }

    stages {
        stage('Checkout Code') {
            steps {
                dir("${WORK_DIR}") {
                    git branch: 'main', url: 'https://github.com/satyanarayana-24/Indie_Gems_Portal.git'
                }
            }
        }
        stage('Install Docker') {
    steps {
        sh '''
        # Check if docker exists
        if ! command -v docker &> /dev/null
        then
            echo "Docker not found. Installing Docker..."

            sudo apt update
            sudo apt install docker.io -y

            sudo systemctl start docker
            sudo systemctl enable docker

            // sudo usermod -aG docker jenkins
        else
            echo "Docker already installed"
        fi

        docker --version
        '''
    }
}
        // stage('Build Docker Image') {
        //     steps {
        //         dir("${WORK_DIR}") {
        //             sh '''
        //                 // docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true
        //                 // docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${WORK_DIR}/Dockerfile ${WORK_DIR}
        //                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
        //             '''
        //         }
        //     }
        // }
        stage('Build Docker Image') {
    steps {
        dir("${WORK_DIR}") {
            sh '''
                # Remove old image if exists
                docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true

                # Build new image
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

                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    """
                }
            }
        }

         stage('Push Image to DockerHub') {
            steps {
                sh """
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }


        stage('Run Docker Container') {
    steps {
        dir("${WORK_DIR}") {
            sh '''
                docker rm -f ${CONTAINER_NAME} || true

                docker run -d \
                -p ${PORT}:${CONTAINER_PORT} \
                --name ${CONTAINER_NAME} \
                ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
            '''
        }
    }
}

        // stage('Run Docker Container') {
        //     steps {
        //         dir("${WORK_DIR}") {
        //             sh '''
        //                 docker rm -f ${CONTAINER_NAME} || true
        //                 // docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${CONTAINER_PORT} \ ${IMAGE_NAME}:${IMAGE_TAG}
        //                   docker run -d \
        //         -p ${HOST_PORT}:${CONTAINER_PORT} \
        //         --name ${CONTAINER_NAME} \
        //         ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
        //             '''
        //         }
        //     }
    

    // post {
    //     always {
    //         echo "Pipeline finished! Check http://13.204.85.107:3000"
    //     }
    }
}





// original one is below
// pipeline {
//     agent any

//     environment {
//         WORK_DIR = "/var/lib/jenkins/workspace/Game"
//         IMAGE_NAME = "indie-gems"
//         IMAGE_TAG  = "latest"
//         CONTAINER_NAME = "indie-gems-container"
//         PORT = "50008j"   // External port for app
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     git branch: 'main', url: 'https://github.com/SukeshKaicharla/Indie_Gems_Portal.git'
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         docker rmi -f ${IMAGE_NAME}:${IMAGE_TAG} || true
//                         docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${WORK_DIR}/Dockerfile ${WORK_DIR}
//                     '''
//                 }
//             }
//         }

//         stage('Run Docker Container') {
//             steps {
//                 dir("${WORK_DIR}") {
//                     sh '''
//                         docker rm -f ${CONTAINER_NAME} || true
//                         docker run -d --name ${CONTAINER_NAME} -p ${PORT}:80 ${IMAGE_NAME}:${IMAGE_TAG}
//                     '''
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             echo "Pipeline finished! Check http://13.204.85.107:3000"
//         }
//     }
// }

