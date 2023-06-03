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
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    def dockerCmd = 'docker run -p 3080:80 -d chinmayapradhan/react-nodejs-app:lts'
                    sshagent(['ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@18.221.54.209 ${dockerCmd}"
                    }
                }
            }
        }
    }
}
