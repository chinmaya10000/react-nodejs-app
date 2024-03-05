pipeline{
    agent any

    environment {
        GITHUB_CREDENTIALS = credentials('github-token')
        DOCKER_IMAGE_NAME = "chinmayapradhan/react-nodejs-app"
        VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
        GITHUB_REPO_URL = "https://github.com/chinmaya10000/react-nodejs-app.git"
    }

    stages {
        stage('Checkout from Git') {
            steps {
                script {
                    echo "Clone the repo.."
                    git 'https://github.com/chinmaya10000/react-nodejs-app.git'
                }
            }
        }
        stage('Build and push Image') {
            steps {
                script {
                    echo "build and push the docker image.."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t ${DOCKER_IMAGE_NAME}:${VERSION} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${VERSION}"
                    }
                }
            }
        }
        stage('Clone/Pull Repo') {
            steps {
                script {
                    if (fileExists('react-nodejs-app')) {
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
