pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'docker'       // ID credenziali Docker in Jenkins
        K8S_TOKEN = credentials('k8s-token')   // ID credenziali token Kubernetes in Jenkins
        K8S_API_SERVER = 'https://<CLUSTER_API>' // Inserisci l'API server del tuo cluster
        NAMESPACE = 'formazione-sou'
        RELEASE_NAME = 'flask-app-release'
        CHART_PATH = 'progettostep2/chart'     // Path relativo del chart Helm
        REGISTRY = 'gabrisource/flask-app-example' // Docker Hub repo
    }

    stages {
        stage('Set Docker Tag') {
            steps {
                script {
                    def gitBranch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    def gitTag = sh(script: "git describe --tags --exact-match || echo ''", returnStdout: true).trim()
                    def gitCommit = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()

                    if (gitTag) {
                        dockerTag = gitTag
                    } else if (gitBranch == 'main') {
                        dockerTag = 'latest'
                    } else {
                        def sanitizedBranch = gitBranch.replaceAll(/[\\/]/, '-')
                        dockerTag = "${sanitizedBranch}-${gitCommit}"
                    }

                    echo "Docker tag will be: ${dockerTag}"
                    env.DOCKER_TAG = dockerTag
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${REGISTRY}:${DOCKER_TAG}", "-f progettostep2/Dockerfile progettostep2")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKERHUB_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                script {
                    def exists = sh(script: """kubectl --server=${K8S_API_SERVER} \
                        --token=${K8S_TOKEN} --insecure-skip-tls-verify get namespace ${NAMESPACE} --ignore-not-found""",
                        returnStatus: true) == 0
                    if (!exists) {
                        echo "Namespace ${NAMESPACE} non esiste. Lo creo."
                        sh "kubectl --server=${K8S_API_SERVER} --token=${K8S_TOKEN} --insecure-skip-tls-verify create namespace ${NAMESPACE}"
                    } else {
                        echo "Namespace ${NAMESPACE} già esistente."
                    }
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                script {
                    sh """
                    helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                        --namespace ${NAMESPACE} \
                        --set image.repository=${REGISTRY} \
                        --set image.tag=${DOCKER_TAG} \
                        --kube-apiserver=${K8S_API_SERVER} \
                        --kube-token=${K8S_TOKEN} \
                        --insecure-skip-tls-verify
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Deploy fallito. Controlla i log."
        }
    }
}


