pipeline {
  agent any
  stages {
    stage('Setup Docker') {
      steps {
        sh 'cat $DOCKER_HOST_CA > ca.pem'
        sh 'cat $DOCKER_HOST_KEY > key.pem'
        sh 'cat $DOCKER_HOST_CERT > cert.pem'
      }
    }
    stage('Export Version & Name') {
      steps {
        sh 'export VERSION=$(cat package.json | jq -r ".version") && echo $VERSION > VERSION'
        sh 'export NAME=$(cat package.json | jq -r ".name" | tr "[:upper:]" "[:lower:]") && echo $NAME > NAME'
      }
    }
    stage('Build Image') {
      steps {
        sh 'docker $(echo $DOCKER_OPTS) build -t $DOCKER_SERVER/$(cat NAME):$(cat VERSION) .'
      }
    }
    stage('Push Image') {
      steps {
        sh 'docker $(echo $DOCKER_OPTS) login $DOCKER_SERVER -u=$DOCKER_USERNAME -p=$DOCKER_PASSWORD'
        sh 'docker $(echo $DOCKER_OPTS) push $DOCKER_SERVER/$(cat NAME):$(cat VERSION)'
      }
    }
    stage('Deploy') {
      steps {
        sh '''curl --fail --insecure --request "POST" --url "api.speedops.com/v1/deployer/kubernetes/deploy" --header "content-type: application/json" --data '{"deployment":{"name":"'"$(cat NAME)"'","version":"'"$(cat VERSION)"'","docker_server":"'"$DOCKER_SERVER"'"},"config":{"host":"'"$KUBE_URL"'","token":"'"$KUBE_TOKEN"'"}}'
'''
      }
    }
  }
  environment {
    KUBE_URL = credentials('KUBE_URL')
    KUBE_TOKEN = credentials('KUBE_TOKEN')
    DOCKER_HOST_CA = credentials('DOCKER_HOST_CA')
    DOCKER_HOST_CERT = credentials('DOCKER_HOST_CERT')
    DOCKER_HOST_KEY = credentials('DOCKER_HOST_KEY')
    DOCKER_SERVER = credentials('DOCKER_SERVER')
    DOCKER_USERNAME = credentials('DOCKER_USERNAME')
    DOCKER_PASSWORD = credentials('DOCKER_PASSWORD')
  }
}