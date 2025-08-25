pipeline {
    agent any

    environment {
        KUBECONFIG_PATH = "${WORKSPACE}/kubeconfig"
        KUBE_TOKEN = credentials('kube-token')       // ID della credenziale token in Jenkins
        KUBE_SERVER = credentials('kube-server')     // URL API server Kubernetes
        KUBE_CA_BASE64 = credentials('kube-ca-base64') // Certificato CA base64
    }

    stages {
        stage('Setup Kubeconfig') {
            steps {
                script {
                    // Scrive il kubeconfig nel workspace
                    sh """
                    mkdir -p ${WORKSPACE}
                    cat > ${KUBECONFIG_PATH} <<EOF
apiVersion: v1
kind: Config
clusters:
- name: cluster
  cluster:
    server: ${KUBE_SERVER}
    certificate-authority-data: ${KUBE_CA_BASE64}
contexts:
- name: jenkins-context
  context:
    cluster: cluster
    user: jenkins
    namespace: default
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

        stage('Ensure Namespace') {
            steps {
                script {
                    sh """
                    kubectl --kubeconfig=${KUBECONFIG_PATH} get namespace formazione-sou || \
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
            // Pulisce il kubeconfig dal workspace
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
