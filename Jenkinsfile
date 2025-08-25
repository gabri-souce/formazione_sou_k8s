pipeline {
    agent any
    environment {
        KUBECONFIG_PATH = "/home/jenkins/.kube/config"
    }
    stages {
        stage('Ensure Namespace') {
            steps {
                script {
                    sh "kubectl --kubeconfig=${KUBECONFIG_PATH} get namespace formazione-sou --ignore-not-found || kubectl --kubeconfig=${KUBECONFIG_PATH} create namespace formazione-sou"
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                script {
                    sh """
                    helm upgrade --install formazione-sou-release charts/hello-node \
                        --namespace formazione-sou \
                        --kubeconfig ${KUBECONFIG_PATH} \
                        --create-namespace \
                        --set image.repository=gabrisource/step4 \
                        --set image.tag=latest
                    """
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

