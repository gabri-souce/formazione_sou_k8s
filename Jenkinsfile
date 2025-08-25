pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REPO = "gabrisource/flask-app-example"
        DOCKER_CREDENTIALS_ID = "docker"
        K8S_SERVER = "https://192.168.3.17:6443"
        K8S_TOKEN_CREDENTIALS_ID = "kube-token"
        NAMESPACE = "formazione-sou"
        HELM_RELEASE = "flask-app"
        HELM_CHART = "./helm/flask-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_REPO}:${GIT_COMMIT}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS_ID) {
                            dockerImage.push()
                        }
                    }
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                withCredentials([string(credentialsId: K8S_TOKEN_CREDENTIALS_ID, variable: 'K8S_TOKEN')]) {
                    script {
                        sh """
                        kubectl --server=${K8S_SERVER} --token=${K8S_TOKEN} --insecure-skip-tls-verify get namespace ${NAMESPACE} || \
                        kubectl --server=${K8S_SERVER} --token=${K8S_TOKEN} --insecure-skip-tls-verify create namespace ${NAMESPACE}
                        """
                    }
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                withCredentials([string(credentialsId: K8S_TOKEN_CREDENTIALS_ID, variable: 'K8S_TOKEN')]) {
                    script {
                        sh """
                        helm upgrade --install ${HELM_RELEASE} ${HELM_CHART} \
                        --namespace ${NAMESPACE} \
                        --set image.repository=${DOCKER_REPO} \
                        --set image.tag=${GIT_COMMIT} \
                        --kube-apiserver=${K8S_SERVER} \
                        --kube-token=${K8S_TOKEN} \
                        --kube-insecure-skip-tls-verify
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deploy completato con successo!'
        }
        failure {
            echo 'Deploy fallito. Controlla i log.'
        }
    }
}


