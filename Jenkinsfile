pipeline{
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-repo')
        GITHUB_CREDENTIALS = credentials('github-token')
        DOCKER_IMAGE_NAME = "chinmayapradhan/react-nodejs-app"
        GITHUB_REPO_URL = "https://github.com/chinmaya10000/react-nodejs-app.git"
    }

    stages {
        stage('Checkout from Git') {
            steps {
                script {
                    echo "Clone the repo.."
                    git branch: 'master', url: 'https://github.com/chinmaya10000/react-nodejs-app.git'
                }
            }
        }
        stage('Build and push Image') {
            steps {
                script {
                    echo "build and push the docker image.."

                    def dockerImageVersion = "v${env.BUILD_NUMBER}"

                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS) {
                        def dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${dockerImageVersion}")
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Clone/Pull Repo') {
            steps {
                script {
                    if (fileExists('DevSecOps-Project')) {
                        echo 'Cloned repo already exists - Pulling latest changes'

                        dir("DevSecOps-Project") {
                            sh 'git pull'
                        }
                    }
                    else {
                        echo 'Repo does not exists - Cloning the repo'
                        sh 'git clone -b master https://github.com/chinmaya10000/react-nodejs-app.git'
                    }
                }
            }
        }
        stage('Update deployment.yaml') {
            steps {
                script {
                    def newImageTag = sh(script: 'docker inspect --format=\'{{.Id}}\' $DOCKER_IMAGE_NAME', returnStdout: true).trim()
                    sh "sed -i 's|image: $DOCKER_IMAGE_NAME.*|image: $DOCKER_IMAGE_NAME:$newImageTag|' kubernetes-manifest/my-app-deployment.yaml"
                }
            }
        }
        stage('Push Changes to GitHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'GITHUB_CREDENTIALS', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh 'git config --global user.name "jenkins"'
                        sh 'git config --global user.email "jenkins"@gmail.com'
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/chinmaya10000/react-nodejs-app.git"
                        sh 'git add kubernetes-manifest/my-app-deployment.yaml'
                        sh 'git commit -m "Updated image version for Build - $dockerImageVersion"'
                        sh 'git push origin HEAD:master'
                    }
                }
            }
        }
    }
}    
