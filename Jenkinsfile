def nameSpace="cloudbees-core"
pipeline {
  agent {
    kubernetes {
      label 'kubectl'
      yaml """
kind: Pod
metadata:
  name: kubectl
spec:
  serviceAccountName: cjoc
  containers:
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    resources:
      requests:
        memory: "500Mi"
    command:
    - cat
    tty: true 
      """
    }
  }
  options { 
    buildDiscarder(logRotator(numToKeepStr: '10'))
    skipDefaultCheckout()
  }
  triggers {
    eventTrigger jmespathQuery("ref=='refs/heads/main' && repository.name=='cloudbees-ci-config-bundle'")
  }
  stages {
    stage('Update Config Bundle') {
      when {
        beforeAgent true
        triggeredBy 'EventTriggerCause'
      }
      environment {
        GITHUB_ORG_LOGIN="${currentBuild.getBuildCauses()[0].event.repository.organization}"
      }
      steps {
        echo "GitHub Org name: ${GITHUB_ORG_LOGIN}" 
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: "https://github.com/${GITHUB_ORG_LOGIN}/cloudbees-ci-config-bundle.git"]]])
        container('kubectl') {
          sh "mkdir -p ${GITHUB_ORG_LOGIN}"
          sh "cp *.yaml ${GITHUB_ORG_LOGIN}"
          sh "kubectl cp --namespace ${nameSpace} ${GITHUB_ORG_LOGIN} cjoc-0:/var/jenkins_home/jcasc-bundles-store/ -c jenkins"
        }
      }
    }
  }
}
