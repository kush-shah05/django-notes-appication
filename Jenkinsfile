pipeline {
    agent { label 'dev' }
    environment {
        IMAGE_NAME = "notes_app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage("code") {
            steps {
                echo "Cloning repository..."
                git url: "https://github.com/kush-shah05/django-notes-appication.git", branch: "main"
            }
        }

        stage("build") {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage("push to dockerhub") {
            steps {
                echo "Pushing Docker image to DockerHub..."
                withCredentials([usernamePassword(
                    credentialsId: "dockerhubCred",
                    usernameVariable: "dockerHubUser",
                    passwordVariable: "dockerHubPass")]) {
                        
                    sh 'echo $dockerHubPass | docker login -u $dockerHubUser --password-stdin'
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${dockerHubUser}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${dockerHubUser}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage("deploy") {
            steps {
                echo "Deploying container from DockerHub image...."
                withCredentials([usernamePassword(
                    credentialsId: "dockerhubCred",
                    usernameVariable: "dockerHubUser",
                    passwordVariable: "dockerHubPass")]) {
                        
                    sh "docker image rm ${dockerHubUser}/${IMAGE_NAME}:${IMAGE_TAG} || true"
                    sh "docker pull ${dockerHubUser}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "IMAGE_TAG=${IMAGE_TAG} docker compose -f docker-compose.yml up -d"
                }
            }
        }
    }
}
