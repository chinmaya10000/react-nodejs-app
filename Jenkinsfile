pipeline {
    
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = 'chinmayapradhan/react-nodejs-app'
        IMAGE_VERSION = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Build and push Image') {
            steps {
                script {
                    echo "build and push the docker image.."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t ${DOCKER_IMAGE_NAME}:${IMAGE_VERSION} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${IMAGE_VERSION}"
                    }
                }
            }
        }
        stage('Push Deployment YAML to GitHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                          git config --global user.name "chinmaya10000"
                          git config --global user.email "chinmayapradhan10000@gmail.com"
                          git remote set-url origin https://${GITHUB_TOKEN}@github.com/chinmaya10000/react-nodejs-app.git
                          sed -i "s#chinmayapradhan.*#${DOCKER_IMAGE_NAME}:${IMAGE_VERSION}#g" kubernetes-manifest/my-app-deployment.yaml
                          git add kubernetes-manifest/my-app-deployment.yaml
                          git commit -m "Updated image version for Build - $IMAGE_VERSION"
                          git push origin HEAD:master
                        '''
                    }
                }
            }
        }
    }
}