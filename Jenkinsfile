pipeline {
    agent any

    environment {
        registryCredential = 'dockerhub-cred'
        NAMESPACE = 'formazione-sou'
        RELEASE_NAME = 'flask-app-example'
        CHART_PATH = 'charts/flask-app-example'
        KUBECONFIG = '/home/jenkins/.kube/config'
    }

    stages {
        stage('Set Docker Tag and Registry') {
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

                    env.registry = "gab/flask-app-example"
                    echo "Docker registry set to: ${env.registry}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${env.registry}:${env.DOCKER_TAG}", "-f progettostep2/Dockerfile progettostep2")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Ensure Namespace') {
            steps {
                script {
                    def exists = sh(script: "kubectl --kubeconfig=${KUBECONFIG} get namespace ${NAMESPACE} --ignore-not-found", returnStatus: true) == 0
                    if (!exists) {
                        echo "Namespace ${NAMESPACE} non esiste. Lo creo."
                        sh "kubectl --kubeconfig=${KUBECONFIG} create namespace ${NAMESPACE}"
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
                        --set image.repository=${env.registry} \
                        --set image.tag=${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Check Deployment Best Practices') {
            steps {
                script {
                    def deployments = sh(script: "kubectl --kubeconfig=${KUBECONFIG} -n ${NAMESPACE} get deployments -o jsonpath='{.items[*].metadata.name}'", returnStdout: true).trim().split()
                    
                    if (deployments.size() == 0) {
                        error "Nessun Deployment trovato nel namespace ${NAMESPACE}"
                    }

                    deployments.each { dep ->
                        echo "Controllo Deployment: ${dep}"

                        def json = sh(script: "kubectl --kubeconfig=${KUBECONFIG} -n ${NAMESPACE} get deployment ${dep} -o json", returnStdout: true)
                        def data = readJSON text: json
                        def containers = data.spec.template.spec.containers

                        containers.each { container ->
                            if (!container.readinessProbe) {
                                error "Manca ReadinessProbe nel container ${container.name} del deployment ${dep}"
                            }
                            if (!container.livenessProbe) {
                                error "Manca LivenessProbe nel container ${container.name} del deployment ${dep}"
                            }
                            if (!container.resources || !container.resources.limits || !container.resources.requests) {
                                error "Manca limits/requests nel container ${container.name} del deployment ${dep}"
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deploy completato con successo e best practices verificate!"
        }
        failure {
            echo "Deploy fallito. Controlla i log per errori nelle best practices."
        }
    }
}
