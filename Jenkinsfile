pipeline {
  agent any
  environment {
    ORG = 'kypseli'
    APP_NAME = 'docker-client'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    DOCKER_REGISTRY_ORG = 'technologists'
  }
  stages {
    stage('CI Build and push snapshot') {
      when {
        branch 'PR-*'
      }
      environment {
        PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
        HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
      }
      steps {
        sh "skaffold version"
        sh "export VERSION=$PREVIEW_VERSION && skaffold build -f skaffold.yaml"
      }
    }
    stage('Build Release') {
      when {
        branch 'master'
      }
      steps {
        git 'https://github.com/kypseli/docker-client.git'
        sh "jx step next-version --use-git-tag-only --tag"
        sh "export VERSION=`cat VERSION` && skaffold build -f skaffold.yaml"
      }
    }
    stage('Promote to Environments') {
      when {
        branch 'master'
      }
      steps {
        sh "jx step changelog --version v\$(cat VERSION)"
      }
    }
  }
}
