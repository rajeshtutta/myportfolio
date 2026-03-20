pipeline {
    agent any

    environment {
        GIT_REPO       = "https://github.com/rajeshtutta/myportfolio.git"
        GIT_BRANCH     = "main"

        DOCKERHUB_USER = "rajeshtutta123"
        IMAGE_NAME     = "rajeshtutta123/rajeshportfolio:6"
        IMAGE_TAG      = "${BUILD_NUMBER}"

        DOCKER_CREDS   = "dockerhub-cred"

        CONTAINER_NAME = "rajeshportfolio"
        HOST_PORT      = "2005"
        CONTAINER_PORT = "8080"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build WAR File') {
            steps {
                // Build WAR file with fixed name (from pom.xml finalName)
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
    steps {
        sh '''
        if [ ! -f target/sample-webapp.war ]; then
            echo "ERROR: WAR file target/sample-webapp.war not found!"
            exit 1
        fi
        docker build -t rajeshtutta123/rajeshportfolio:6 .
        '''
    }
}

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true

                docker run -d \
                    -p ${HOST_PORT}:${CONTAINER_PORT} \
                    --name ${CONTAINER_NAME} \
                    ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }
}
