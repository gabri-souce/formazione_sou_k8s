pipeline {
    agent any
    environment {
        // Percorso temporaneo del kubeconfig nel workspace
        KUBECONFIG_PATH = "${WORKSPACE}/kubeconfig"
    }
    stages {
        stage('Setup Kubeconfig') {
            steps {
                withCredentials([
                    string(credentialsId: 'kube-token', variable: 'KUBE_TOKEN'),
                    string(credentialsId: 'kube-ca-base64', variable: 'KUBE_CA_BASE64'),
                    string(credentialsId: 'kube-server', variable: 'KUBE_SERVER')
                ]) {
                    script {
                        sh """
                        mkdir -p \$(dirname ${KUBECONFIG_PATH})
                        cat > ${KUBECONFIG_PATH} <<EOF
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${KUBE_CA_BASE64}
    server: ${KUBE_SERVER}
  name: k8s-cluster
contexts:
- context:
    cluster: k8s-cluster
    user: jenkins
  name: jenkins-context
current-context: jenkins-context
users:
- name: jenkins
  user:
    token: ${KUBE_TOKEN}
EOF
                        """
                    }
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                script {
                    sh """
                    kubectl --kubeconfig=${KUBECONFIG_PATH} get namespace formazione-sou --ignore-not-found || \
                    kubectl --kubeconfig=${KUBECONFIG_PATH} create namespace formazione-sou
                    """
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
        always {
            // Pulizia del kubeconfig temporaneo
            sh "rm -f ${KUBECONFIG_PATH}"
        }
        success {
            echo "Deploy completato con successo!"
        }
        failure {
            echo "Deploy fallito. Controlla i log."
        }
    }
}


