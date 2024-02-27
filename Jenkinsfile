pipeline {
    agent any
    tools {
        maven 'MAVEN'
    }
    environment {
        DOCKER_TAG = getVersion()
    }
    stages {
        stage('Git Checkout') {
            steps {
                git credentialsId: 'github', url: 'https://github.com/aniketmore620/Banking-project.git'
            }
        }
        stage('Maven Package') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build Image') {
            steps {
                sh 'docker build . -t moreaniket/banking:${DOCKER_TAG}'
                sh 'docker build . -t moreaniket/banking:latest'
            }
        }
        stage('Docker Hub Push') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'Dockerhubpwd')]) {
                    sh 'docker login -u moreaniket -p ${Dockerhubpwd}'
                }
                sh 'docker push moreaniket/banking:${DOCKER_TAG}'
                sh 'docker push moreaniket/banking:latest'
            }
        }
        stage('Deploy') {
            steps {
                ansiblePlaybook credentialsId: 'test-server',
                                disableHostKeyChecking: true,
                                extras: "-e DOCKER_TAG=${DOCKER_TAG}",
                                installation: 'ansible',
                                inventory: 'ans-inventory.inv',
                                playbook: 'ansible-playbook.yml',
                                vaultTmpPath: ''
            }
        }
    }
}

def getVersion() {
    def commitHash = sh returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash.trim()
}
