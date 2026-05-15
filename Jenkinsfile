pipeline {
    agent any

    environment {
        IMAGE_NAME = "portfolio"
        TAG = "build-${BUILD_NUMBER}"
        PUBLIC_IP = "40.192.36.35"
    }

    stages {

        stage('Git Clone') {
            steps {
                git 'https://github.com/VigneshHubs/Portfolio.git'
            }
        }

        stage('Upload to S3') {
            steps {
                sh 'aws s3 cp . s3://vignesh-devops-artifacts --recursive --exclude ".git/*"'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Docker Push') {
            steps {

                withDockerRegistry(
                    [credentialsId: 'fb37fb36-7cbc-4621-8edd-c7ef6536ebdf',
                     url: 'https://index.docker.io/v1/']) {

                    sh "docker tag ${IMAGE_NAME}:${TAG} vigneshhub/${IMAGE_NAME}:${TAG}"

                    sh "docker push vigneshhub/${IMAGE_NAME}:${TAG}"
                    
                    sh "docker ps"
                }
            }
        }

        stage('Docker Run') {
            steps {

                sh 'docker rm -f portfolio-container >/dev/null 2>&1 || true'

                sh "docker run -d --name portfolio-container -p 8989:80 vigneshhub/${IMAGE_NAME}:${TAG}"
            }
        }
    }

    post {

        success {

            emailext(
                to: 'siddardha070@gmail.com, vignesh.s.kumar.in@gmail.com',

                subject: "Build Success - ${JOB_NAME} #${BUILD_NUMBER}",

                body: """
Application deployed successfully.

Job Name: ${JOB_NAME}
Build Number: ${BUILD_NUMBER}
Docker Image:
vigneshhub/${IMAGE_NAME}:${TAG}

Access Application:
http://${PUBLIC_IP}:8989
"""
            )
        }

        failure {

            emailext(
                to: 'siddardha070@gmail.com, vignesh.s.kumar.in@gmail.com',

                subject: "Build Failed - ${JOB_NAME} #${BUILD_NUMBER}",

                body: """
Build failed.

Job Name: ${JOB_NAME}
Build Number: ${BUILD_NUMBER}
40.192.36.35
Check Jenkins Console Output for errors.
"""
            )
        }
    }
}
