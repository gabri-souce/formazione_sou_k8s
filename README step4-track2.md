-Avvio vm con ensible, con conteiner con jenkins,installo kind in locale,configura config.yaml per permessi,porte e ip,lo copio sul config della vm per poter vedere il mio cluster da vm,creo pipeline che da git effettui helm install su namespace creato ul cluster
-conntrollare presenza di kubectl,helm in ambienti,sto attento a dome metto config (sua per vm che kube)(es vm: kubectl --kubeconfig=/home/vagrant/kind-my-cluster.kubeconfig get svc -n formazione-sou

pipeline {
    agent any

    environment {
        KUBECONFIG = '/home/jenkins/kubeconfig'
        NAMESPACE = 'formazione-sou'
        RELEASE_NAME = 'formazione-sou-release'
        CHART_PATH = 'charts/hello-node'
        GIT_REPO = 'https://github.com/gabri-souce/formazione_sou_k8s.git'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Clone GitHub repo') {
            steps {
                git branch: "${env.GIT_BRANCH}", url: "${env.GIT_REPO}"
            }
        }

        stage('Ensure Namespace') {
            steps {
                script {
                    def nsExists = sh(
                        script: "kubectl --kubeconfig=${env.KUBECONFIG} get namespace ${env.NAMESPACE} --ignore-not-found",
                        returnStatus: true
                    ) == 0

                    if (!nsExists) {
                        echo "Namespace ${env.NAMESPACE} non esiste. Lo creo."
                        sh "kubectl --kubeconfig=${env.KUBECONFIG} create namespace ${env.NAMESPACE}"
                    } else {
                        echo "Namespace ${env.NAMESPACE} già esistente."
                    }
                }
            }
        }

        stage('Helm Install/Upgrade') {
            steps {
                script {
                    sh """
                    helm upgrade --install ${env.RELEASE_NAME} ${env.CHART_PATH} \
                        --namespace ${env.NAMESPACE} --kubeconfig ${env.KUBECONFIG} --create-namespace
                    """
                }
            }
        }
    }
}
