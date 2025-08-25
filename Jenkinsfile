pipeline {
    agent any

    environment {
        NAMESPACE = "formazione-sou"
        KUBECONFIG_PATH = "${WORKSPACE}/kubeconfig"
        HELM_RELEASE = "formazione-sou-release"
        HELM_CHART = "charts/hello-node"
        IMAGE_REPO = "gabrisource/step4"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/gabri-souce/formazione_sou_k8s.git']]
                ])
            }
        }

        stage('Setup Kubeconfig') {
            steps {
                withCredentials([
                    string(credentialsId: 'kube-server', variable: 'KUBE_SERVER'),
                    string(credentialsId: 'kube-token', variable: 'KUBE_TOKEN'),
                    string(credentialsId: 'kube-ca-base64', variable: 'KUBE_CA_BASE64')
                ]) {
                    sh """
                    cat <<EOF > ${KUBECONFIG_PATH}
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: \$KUBE_CA_BASE64
    server: \$KUBE_SERVER
  name: jenkins-cluster
contexts:
- context:
    cluster: jenkins-cluster
    user: jenkins
    namespace: ${NAMESPACE}
  name: jenkins-context
current-context: jenkins-context
users:
- name: jenkins
  user:
    token: \$KUBE_TOKEN
EOF
                    """
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                sh """
                if ! kubectl --kubeconfig=${KUBECONFIG_PATH} get namespace ${NAMESPACE} >/dev/null 2>&1; then
                    echo "Namespace ${NAMESPACE} non esiste. Creazione..."
                    kubectl --kubeconfig=${KUBECONFIG_PATH} create namespace ${NAMESPACE}
                else
                    echo "Namespace ${NAMESPACE} già esistente."
                fi
                """
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                sh """
                helm upgrade --install ${HELM_RELEASE} ${HELM_CHART} \
                    --namespace ${NAMESPACE} \
                    --kubeconfig ${KUBECONFIG_PATH} \
                    --create-namespace \
                    --set image.repository=${IMAGE_REPO} \
                    --set image.tag=${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
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


