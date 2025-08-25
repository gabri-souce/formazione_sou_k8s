
pipeline {
    agent any

    environment {
        KUBECONFIG = "${WORKSPACE}/kubeconfig"
    }

    stages {
        stage('Setup Kubeconfig') {
            steps {
                withCredentials([
                    string(credentialsId: 'kube-token', variable: 'KUBE_TOKEN'),
                    string(credentialsId: 'kube-ca', variable: 'KUBE_CA_BASE64'),
                    string(credentialsId: 'kube-server', variable: 'KUBE_SERVER')
                ]) {
                    sh """
                        # Decodifica il certificato CA in un file temporaneo
                        echo "$KUBE_CA_BASE64" | base64 -d > /tmp/ca.crt

                        # Configura cluster
                        kubectl config set-cluster mycluster \
                          --server=$KUBE_SERVER \
                          --certificate-authority=/tmp/ca.crt \
                          --embed-certs=true \
                          --kubeconfig=$KUBECONFIG

                        # Configura user con token
                        kubectl config set-credentials jenkins \
                          --token=$KUBE_TOKEN \
                          --kubeconfig=$KUBECONFIG

                        # Configura context con namespace
                        kubectl config set-context jenkins@mycluster \
                          --cluster=mycluster \
                          --user=jenkins \
                          --namespace=formazione-sou \
                          --kubeconfig=$KUBECONFIG

                        kubectl config use-context jenkins@mycluster --kubeconfig=$KUBECONFIG
                    """
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                sh """
                    if ! kubectl --kubeconfig=$KUBECONFIG get namespace formazione-sou > /dev/null 2>&1; then
                        echo "Namespace formazione-sou non esiste. Lo creo."
                        kubectl --kubeconfig=$KUBECONFIG create namespace formazione-sou
                    else
                        echo "Namespace formazione-sou già esistente."
                    fi
                """
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                sh """
                    helm upgrade --install formazione-sou-release ./chart \
                      --namespace formazione-sou \
                      --kubeconfig $KUBECONFIG
                """
            }
        }
    }

    post {
        always {
            sh 'rm -f /tmp/ca.crt'
        }
    }
}
