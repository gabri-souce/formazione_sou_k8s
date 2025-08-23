{\rtf1\ansi\ansicpg1252\cocoartf2709
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\paperw11900\paperh16840\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 pipeline \{\
  agent any\
\
  environment \{\
    registry = 'gabrisource/step4'        // Docker Hub repository\
    registryCredential = 'docker'                  // Jenkins credentials ID\
    dockerTag = ''\
    KUBECONFIG = '/home/jenkins/.kube/config'        // Percorso kubeconfig nel container Jenkins\
    NAMESPACE = 'formazione-sou'                   // Namespace Kubernetes\
    RELEASE_NAME = 'formazione-sou-release'        // Nome release Helm\
    CHART_PATH = 'charts/hello-node'               // Path della chart Helm nel repo\
  \}\
\
  stages \{\
    stage('Clone Git') \{\
      steps \{\
        git branch: 'main', url: 'https://github.com/gabri-souce/formazione_sou_k8s.git'\
      \}\
    \}\
\
    stage('Set Docker Tag') \{\
      steps \{\
        script \{\
          def gitBranch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()\
          def gitTag = sh(script: "git describe --tags --exact-match || echo ''", returnStdout: true).trim()\
          def gitCommit = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()\
\
          if (gitTag) \{\
            dockerTag = gitTag\
          \} else if (gitBranch == 'main') \{\
            dockerTag = 'latest'\
          \} else \{\
            def sanitizedBranch = gitBranch.replaceAll(/[\\\\/]/, '-')\
            dockerTag = "$\{sanitizedBranch\}-$\{gitCommit\}"\
          \}\
\
          echo "Docker tag will be: $\{dockerTag\}"\
          env.DOCKER_TAG = dockerTag\
        \}\
      \}\
    \}\
\
    stage('Build Docker Image') \{\
      steps \{\
        script \{\
          dockerImage = docker.build("$\{registry\}:$\{env.DOCKER_TAG\}", "-f progettostep2/Dockerfile progettostep2")\
        \}\
      \}\
    \}\
\
    stage('Push Docker Image') \{\
      steps \{\
        script \{\
          docker.withRegistry('https://registry.hub.docker.com', registryCredential) \{\
            dockerImage.push()\
          \}\
        \}\
      \}\
    \}\
\
    stage('Ensure Namespace') \{\
      steps \{\
        script \{\
          def exists = sh(script: "kubectl --kubeconfig=$\{KUBECONFIG\} get namespace $\{NAMESPACE\} --ignore-not-found", returnStatus: true) == 0\
          if (!exists) \{\
            echo "Namespace $\{NAMESPACE\} non esiste. Lo creo."\
            sh "kubectl --kubeconfig=$\{KUBECONFIG\} create namespace $\{NAMESPACE\}"\
          \} else \{\
            echo "Namespace $\{NAMESPACE\} gi\'e0 esistente."\
          \}\
        \}\
      \}\
    \}\
\
    stage('Helm Install/Upgrade') \{\
      steps \{\
        script \{\
          sh """\
          helm upgrade --install $\{RELEASE_NAME\} $\{CHART_PATH\} \\\
            --namespace $\{NAMESPACE\} --kubeconfig $\{KUBECONFIG\} --create-namespace \\\
            --set image.repository=$\{registry\} \\\
            --set image.tag=$\{DOCKER_TAG\}\
          """\
        \}\
      \}\
    \}\
  \}\
\}}