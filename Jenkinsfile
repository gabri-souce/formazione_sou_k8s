pipeline {
    agent any

    environment {
        K8S_SERVER = "https://192.168.3.17:6443"
        K8S_NAMESPACE = "formazione-sou"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def commit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    def dockerImage = docker.build("gabrisource/flask-app-example:${commit}", "./progettostep2")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-cred', toolName: 'docker') {
                    script {
                        sh "docker push gabrisource/flask-app-example:$(git rev-parse HEAD)"
                    }
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                withCredentials([string(credentialsId: 'kube-token', variable: 'K8S_TOKEN')]) {
                    script {
                        sh """
                        kubectl --server=$K8S_SERVER \
                            --token=$K8S_TOKEN \
                            --insecure-skip-tls-verify \
                            get namespace $K8S_NAMESPACE || \
                        kubectl --server=$K8S_SERVER \
                            --token=$K8S_TOKEN \
                            --insecure-skip-tls-verify \
                            create namespace $K8S_NAMESPACE
                        """
                    }
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                withCredentials([string(credentialsId: 'kube-token', variable: 'K8S_TOKEN')]) {
                    script {
                        sh """
                        helm upgrade --install flask-app ./helm-chart \
                            --namespace $K8S_NAMESPACE \
                            --set image.repository=gabrisource/flask-app-example \
                            --set image.tag=$(git rev-parse HEAD) \
                            --kube-token=$K8S_TOKEN \
                            --kube-apiserver=$K8S_SERVER \
                            --kube-insecure-skip-tls-verify
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "Deploy fallito. Controlla i log."
        }
    }
}



