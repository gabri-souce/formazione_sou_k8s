pipeline {
    agent any

    environment {
        KUBE_CONFIG = '/var/jenkins_home/workspace/step4/kubeconfig'
        HELM_RELEASE = 'formazione-sou-release'
        HELM_CHART = 'charts/hello-node'
        NAMESPACE = 'formazione-sou'
        IMAGE_REPO = 'gabrisource/step4'
        IMAGE_TAG = 'latest'
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
                    string(credentialsId: 'kube-ca-base64', variable: 'KUBE_CA_BASE64'),
                    string(credentialsId: 'kube-token', variable: 'KUBE_TOKEN'),
                    string(credentialsId: 'kube-server', variable: 'KUBE_SERVER')
                ]) {
                    sh '''
                        mkdir -p $(dirname ${KUBE_CONFIG})
                        cat <<EOF > ${KUBE_CONFIG}
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${KUBE_CA_BASE64}
    server: ${KUBE_SERVER}
  name: mycluster
contexts:
- context:
    cluster: mycluster
    namespace: ${NAMESPACE}
    user: jenkins
  name: jenkins@mycluster
current-context: jenkins@mycluster
users:
- name: jenkins
  user:
    token: ${KUBE_TOKEN}
EOF
                    '''
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                sh """
                    helm upgrade --install ${HELM_RELEASE} ${HELM_CHART} \
                        --namespace ${NAMESPACE} \
                        --kubeconfig ${KUBE_CONFIG} \
                        --set image.repository=${IMAGE_REPO} \
                        --set image.tag=${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            sh "rm -f ${KUBE_CONFIG}"
        }
        failure {
            echo 'Deploy fallito. Controlla i log.'
        }
        success {
            echo 'Deploy completato con successo!'
        }
    }
}

