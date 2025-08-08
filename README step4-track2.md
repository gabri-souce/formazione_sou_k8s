Esecuzione Track due dove è stata creata un vm (vagrant) con ansible, e jenkins che gira in un conteiner (dove ho inserito kubectl,docker,helm), nella pipeline è stata configurato il build, il push dell imagine del proprio Docker Hub con impostazione e selezione TAG con la configurazione di Helm.Successivamente  è stato installato un cluster (kind) e automatizzata la creazione di un namespace, effettuato "helm install su namespace creato". Successivamente è stato creato su kind un serviceaccount, eseguito un export del Deployment dell'applicazione Flask installata e che ritorni errore se non presenti nel Deployment i seguenti attributi: Readiness e Liveness Probles, Limits e Requests inoltre è stato generato un token che ho utilizzato per stabilire l' affidabilita tra la mia vm con connteiner è il claster(inserito in file config conteiner), inoltre è stato configurato un ingress controll in modo che chiamando via HTTP http://formazionesou.local si ottenga "hello world".

pipeline {
  agent any

  environment {
    registry = 'gabrisource/step4'        // Docker Hub repository
    registryCredential = 'docker'                  // Jenkins credentials ID
    dockerTag = ''
    KUBECONFIG = '/home/jenkins/.kube/config'        // Percorso kubeconfig nel container Jenkins
    NAMESPACE = 'formazione-sou'                   // Namespace Kubernetes
    RELEASE_NAME = 'formazione-sou-release'        // Nome release Helm
    CHART_PATH = 'charts/hello-node'               // Path della chart Helm nel repo
  }

  stages {
    stage('Clone Git') {
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
          docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
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
            --set image.repository=${registry} \
            --set image.tag=${DOCKER_TAG}
          """
        }
      }
    }
  }
}

