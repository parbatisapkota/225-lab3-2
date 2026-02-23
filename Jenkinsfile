pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'
        DOCKER_IMAGE = 'cithit/sapkotp2'
        IMAGE_TAG = "latest"
        GITHUB_URL = 'https://github.com/parbatisapkota/225-lab3-2.git'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
        stage('Deploy to Dev Environment using NodePort') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'roseaw-225', variable: 'KUBECONFIG')]) {
                        sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                        sh "kubectl apply -f deployment-dev.yaml"
                    }
                }
            }
        }
        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'roseaw-225', variable: 'KUBECONFIG')]) {
                        sh "kubectl get all"
                    }
                }
            }
        }
    }
    post {
        success {
            slackSend([color: "good", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"])
        }
        unstable {
            slackSend([color: "warning", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"])
        }
        failure {
            slackSend([color: "danger", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"])
        }
    }
}
