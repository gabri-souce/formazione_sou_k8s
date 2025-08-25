pipeline {
    agent any

    environment {
        registry = "gabrisource/flask-app-example"
        registryCredential = "docker" // ID delle credenziali Docker in Jenkins
        NAMESPACE = "formazione-sou"
        RELEASE_NAME = "flask-app"
        CHART_PATH = "./progettostep2/chart"
        KUBECONFIG = "/home/jenkins/.kube/config"
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
                    def dockerImage = docker.build("${registry}:${env.DOCKER_TAG}", "-f progettostep2/Dockerfile progettostep2")
                    env.DOCKER_IMAGE = dockerImage.imageName()
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        docker.image(env.DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                script {
                    def exists = sh(script: "kubectl --kubeconfig=${KUBECONFIG} --insecure-skip-tls-verify get namespace ${NAMESPACE} --ignore-not-found", returnStatus: true) == 0
                    if (!exists) {
                        echo "Namespace ${NAMESPACE} non esiste. Lo creo."
                        sh "kubectl --kubeconfig=${KUBECONFIG} --insecure-skip-tls-verify create namespace ${NAMESPACE}"
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
                        --namespace ${NAMESPACE} --kubeconfig ${KUBECONFIG} --create-namespace \
                        --set image.repository=${registry} \
                        --set image.tag=${DOCKER_TAG} \
                        --insecure-skip-tls-verify
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deploy completato con successo!"
        }
        failure {
            echo "Deploy fallito. Controlla i log."
        }
    }
}

