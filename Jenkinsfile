pipeline {
    agent any

    environment {
        registry = "gabrisource/flask-app-example"
        registryCredential = 'docker'      // Docker Hub credential ID
        K8S_TOKEN_CRED = 'kube-token'      // Jenkins secret text con token Kubernetes
        K8S_SERVER = 'https://192.168.3.17:6443'  // API server del tuo cluster
        NAMESPACE = 'formazione-sou'
        RELEASE_NAME = 'flask-app'
        CHART_PATH = './progettostep2/helm-chart'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gabri-souce/formazione_sou_k8s.git'
            }
        }

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
                    dockerImage = docker.build("${registry}:${env.DOCKER_TAG}", "-f progettostep2/Dockerfile progettostep2")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                withCredentials([string(credentialsId: K8S_TOKEN_CRED, variable: 'K8S_TOKEN')]) {
                    script {
                        def exists = sh(script: "kubectl --server=${K8S_SERVER} --token=${K8S_TOKEN} --insecure-skip-tls-verify get namespace ${NAMESPACE} --ignore-not-found", returnStatus: true) == 0
                        if (!exists) {
                            echo "Namespace ${NAMESPACE} non esiste. Lo creo."
                            sh "kubectl --server=${K8S_SERVER} --token=${K8S_TOKEN} --insecure-skip-tls-verify create namespace ${NAMESPACE}"
                        } else {
                            echo "Namespace ${NAMESPACE} già esistente."
                        }
                    }
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                withCredentials([string(credentialsId: K8S_TOKEN_CRED, variable: 'K8S_TOKEN')]) {
                    sh """
                        helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                            --namespace ${NAMESPACE} \
                            --kube-apiserver ${K8S_SERVER} \
                            --kube-token \$K8S_TOKEN \
                            --insecure-skip-tls-verify \
                            --set image.repository=${registry} \
                            --set image.tag=${DOCKER_TAG}
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Deploy fallito. Controlla i log per errori."
        }
    }
}


