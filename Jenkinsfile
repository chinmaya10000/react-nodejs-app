#!/usr/bin/env groovy

pipeline {
    agent any
    stages {
        stage('test') {
            steps {
                script {
                    echo "testing the application..."
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'docker build -t chinmayapradhan/react-nodejs-app:lts .'
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh 'docker push chinmayapradhan/react-nodejs-app:lts'
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    def dockerCmd = 'docker run -p 3000:80 -d chinmayapradhan/react-nodejs-app:lts'
                    sshagent(['ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@13.58.102.138 ${dockerCmd}"
                    }
                }
            }
        }
    }
}
