pipeline {
    agent any

    environment {
        KUBECONFIG_PATH = "${WORKSPACE}/kubeconfig"
        NAMESPACE = "formazione-sou"
        HELM_RELEASE = "formazione-sou-release"
        CHART_PATH = "charts/hello-node"
        IMAGE_REPO = "gabrisource/step4"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gabri-souce/formazione_sou_k8s.git'
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
                        echo \$KUBE_CA_BASE64 | base64 -d > ${KUBECONFIG_PATH}
                        kubectl config set-cluster jenkins-cluster --server=\$KUBE_SERVER --certificate-authority=${KUBECONFIG_PATH} --embed-certs=true
                        kubectl config set-credentials jenkins --token=\$KUBE_TOKEN
                        kubectl config set-context jenkins --cluster=jenkins-cluster --user=jenkins --namespace=${NAMESPACE}
                        kubectl config use-context jenkins
                    """
                }
            }
        }

        stage('Check Namespace') {
            steps {
                script {
                    def nsExists = sh(
                        script: "kubectl get namespace ${NAMESPACE} --kubeconfig ${KUBECONFIG_PATH} --ignore-not-found",
                        returnStdout: true
                    ).trim()

                    if (nsExists == "") {
                        error "Namespace ${NAMESPACE} non trovato! Crealo manualmente prima di eseguire la pipeline."
                    } else {
                        echo "Namespace ${NAMESPACE} esiste. Procedo con Helm."
                    }
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                sh """
                    helm upgrade --install ${HELM_RELEASE} ${CHART_PATH} \
                        --namespace ${NAMESPACE} \
                        --kubeconfig ${KUBECONFIG_PATH} \
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
    }
}

