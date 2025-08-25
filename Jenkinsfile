pipeline {
    agent any

    environment {
        DOCKER_REPO = "gabrisource/flask-app-example"
        K8S_NAMESPACE = "formazione-sou"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gabri-souce/formazione_sou_k8s.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Costruisci l'immagine Docker dalla cartella progettostep2
                    def imageTag = "${DOCKER_REPO}:${GIT_COMMIT}"
                    dockerImage = docker.build(imageTag, "progettostep2")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/']) {
                    script {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                withCredentials([string(credentialsId: 'kube-token', variable: 'K8S_TOKEN')]) {
                    script {
                        sh """
                        kubectl --server=https://192.168.3.17:6443 \
                                --token=$K8S_TOKEN \
                                --insecure-skip-tls-verify \
                                get namespace ${K8S_NAMESPACE} || \
                        kubectl --server=https://192.168.3.17:6443 \
                                --token=$K8S_TOKEN \
                                --insecure-skip-tls-verify \
                                create namespace ${K8S_NAMESPACE}
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
                        helm upgrade --install flask-app-progetto \
                            progettostep2/helm-chart \
                            --namespace ${K8S_NAMESPACE} \
                            --set image.repository=${DOCKER_REPO} \
                            --set image.tag=${GIT_COMMIT} \
                            --kube-token $K8S_TOKEN \
                            --kube-apiserver https://192.168.3.17:6443 \
                            --kube-insecure-skip-tls-verify
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deploy completato con successo!"
        }
        failure {
            echo "Deploy fallito. Controlla i log."
        }
    }
}


