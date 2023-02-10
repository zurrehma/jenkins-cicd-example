// def REGISTRY_URL = 'hub.docker.com'
def OWNER = 'zahid401'
def REPO_NAME = 'jenkins-cicd-example'
def IMAGE_NAME = 'helloworld'

def IMAGE_REGISTRY = "${OWNER}/${REPO_NAME}/${IMAGE_NAME}"
def IMAGE_BRANCH_TAG = "${IMAGE_REGISTRY}"

def REGISTRY_CREDENTIALS = 'docker-creds'
def DOCKER_HOST_VALUE = 'tcp://dind.default:2375'

def DOCKER_POD = """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:19.03.6
    command:
    - cat
    tty: true
    env:
    - name: DOCKER_HOST
      value: ${DOCKER_HOST_VALUE}
"""

pipeline {
  agent any
  stages {
    stage('Run Docker') {
      agent { kubernetes label: 'docker', yaml: "${DOCKER_POD}" }
      stages {
        stage('Build Docker Image') {
          steps {
            container('docker') {
              sh "docker build -t ${IMAGE_BRANCH_TAG}.${env.GIT_COMMIT[0..6]} ."
            }
          }
        }
        stage('Push Image to Registry') {
          steps {
            container('docker') {
              withCredentials([
                usernamePassword(
                  credentialsId: "${REGISTRY_CREDENTIALS}",
                  usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASS'
                )
              ]) {
                sh """
                echo ${REGISTRY_PASS} | docker login -u ${REGISTRY_USER} --password-stdin
                docker push ${IMAGE_BRANCH_TAG}.${env.GIT_COMMIT[0..6]}
                docker tag ${IMAGE_BRANCH_TAG}.${env.GIT_COMMIT[0..6]} ${IMAGE_BRANCH_TAG}:${env.GIT_COMMIT[0..6]}
                docker push ${IMAGE_BRANCH_TAG}:${env.GIT_COMMIT[0..6]}
                """
              }
            }
          }
        }
      }
    }
  }
}
