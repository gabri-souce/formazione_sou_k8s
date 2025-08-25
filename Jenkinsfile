pipeline {
    agent any

    environment {
        KUBE_NAMESPACE = 'formazione-sou'
        KUBECONFIG_PATH = "${WORKSPACE}/kubeconfig"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/gabri-souce/formazione_sou_k8s.git', branch: 'main'
            }
        }

        stage('Setup Kubeconfig') {
            steps {
                withCredentials([
                    string(credentialsId: 'kube-token', variable: 'KUBE_TOKEN'),
                    string(credentialsId: 'kube-ca-base64', variable: 'KUBE_CA')
                ]) {
                    script {
                        // Sostituisci con il server API corretto della tua VM/cluster
                        def kubeServer = 'https://192.168.33.10:6443'

                        sh """
                        mkdir -p ${WORKSPACE}
                        cat > ${KUBECONFIG_PATH} <<EOF
apiVersion: v1
kind: Config
clusters:
- cluster:
    server: ${kubeServer}
    certificate-authority-data: ${KUBE_CA}
  name: k8s-cluster
contexts:
- context:
    cluster: k8s-cluster
    user: jenkins
    namespace: ${KUBE_NAMESPACE}
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
                    kubectl --kubeconfig=${KUBECONFIG_PATH} get namespace ${KUBE_NAMESPACE} --ignore-not-found || \
                    kubectl --kubeconfig=${KUBECONFIG_PATH} create namespace ${KUBE_NAMESPACE}
                    """
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                sh """
                helm upgrade --install formazione-sou-release charts/hello-node \
                --namespace ${KUBE_NAMESPACE} \
                --kubeconfig ${KUBECONFIG_PATH} \
                --create-namespace \
                --set image.repository=gabrisource/step4 \
                --set image.tag=latest
                """
            }
        }
    }

    post {
        always {
            sh "rm -f ${KUBECONFIG_PATH}"
            echo "Deploy completato o fallito, kubeconfig rimosso."
        }
    }
}
