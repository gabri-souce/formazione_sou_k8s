pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')  // Jenkins Credential ID
        KUBE_TOKEN = credentials('kube-token')           // Jenkins Credential ID del Service Account K8s
        KUBE_SERVER = 'https://192.168.56.20:6443'      // Indirizzo API server K8s
        NAMESPACE = 'formazione-sou'
        CHART_PATH = 'charts/flask-app'
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
                    def tag = 'latest'
                    if(env.GIT_TAG_NAME) {
                        tag = env.GIT_TAG_NAME
                    } else if(env.BRANCH_NAME == 'develop') {
                        tag = "develop-${env.GIT_COMMIT.substring(0,7)}"
                    }

                    echo "Building Docker image with tag: ${tag}"

                    sh '''
                        docker build -t ${DOCKERHUB_CREDENTIALS_USR}/flask-app:${tag} .
                        echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                        docker push ${DOCKERHUB_CREDENTIALS_USR}/flask-app:${tag}
                    '''
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                script {
                    sh """
                        kubectl --token=${KUBE_TOKEN} --server=${KUBE_SERVER} get namespace ${NAMESPACE} --ignore-not-found || \
                        kubectl --token=${KUBE_TOKEN} --server=${KUBE_SERVER} create namespace ${NAMESPACE}
                    """
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                script {
                    sh """
                        helm upgrade --install flask-app ${CHART_PATH} \
                            --namespace ${NAMESPACE} \
                            --set image.repository=${DOCKERHUB_CREDENTIALS_USR}/flask-app \
                            --set image.tag=${env.GIT_TAG_NAME ?: (env.BRANCH_NAME == 'develop' ? "develop-${env.GIT_COMMIT.substring(0,7)}" : "latest")} \
                            --kube-token=${KUBE_TOKEN} \
                            --kube-apiserver=${KUBE_SERVER} \
                            --insecure-skip-tls-verify
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deploy completato con successo!'
        }
        failure {
            echo 'Deploy fallito. Controlla i log.'
        }
    }
}



