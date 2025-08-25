pipeline {
  agent any

  environment {
    registry = 'gabrisource/step4'          // Docker Hub repository
    registryCredential = 'docker'           // Jenkins credentials ID
    dockerTag = ''
    KUBECONFIG = "${env.WORKSPACE}/kubeconfig"  // kubeconfig generato dinamicamente
    NAMESPACE = 'formazione-sou'           // Namespace Kubernetes
    RELEASE_NAME = 'formazione-sou-release' // Nome release Helm
    CHART_PATH = 'charts/hello-node'       // Path della chart Helm nel repo
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

    stage('Setup Kubeconfig') {
      steps {
        withCredentials([
          string(credentialsId: 'kube-token', variable: 'KUBE_TOKEN'),
          string(credentialsId: 'kube-ca', variable: 'KUBE_CA'),
          string(credentialsId: 'kube-server', variable: 'KUBE_SERVER')
        ]) {
          sh '''
            echo $KUBE_CA | base64 -d > ca.crt

            kubectl config set-cluster mycluster \
              --server=$KUBE_SERVER \
              --certificate-authority=ca.crt \
              --embed-certs=true \
              --kubeconfig=$KUBECONFIG

            kubectl config set-credentials jenkins \
              --token=$KUBE_TOKEN \
              --kubeconfig=$KUBECONFIG

            kubectl config set-context jenkins@mycluster \
              --cluster=mycluster \
              --user=jenkins \
              --namespace=${NAMESPACE} \
              --kubeconfig=$KUBECONFIG

            kubectl config use-context jenkins@mycluster --kubeconfig=$KUBECONFIG
          '''
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
