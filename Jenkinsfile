pipeline {
    agent any
        //docker { image 'shivam139/jenkins-agent-ubuntu:v2' args '--user root -v /var/run/docker.sock:/var/run/docker.sock' }

    environment {
        IMAGE_TAG = "v${env.BUILD_NUMBER}"
    }

    stages {
        stage('checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/shivampawar777/streamlit_application-demo.git' 
                sh 'echo passed...'
            }
        }
        stage('Build & push docker image') {
            steps {
                echo 'Building the docker image And then pushing image to the dockerHub'
                withCredentials([
                    usernamePassword(credentialsId: "dockerhub",
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                    )]) {
                    sh """
                        cd Python-app
                        docker build -t ${DOCKER_USER}/streamlit_application:${env.IMAGE_TAG} .
                        docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}
                        docker push ${DOCKER_USER}/streamlit_application:${env.IMAGE_TAG}
                    """
                }
            }
        }
        stage('Update deployment manifest') {
            environment {
                GITHUB_EMAIL = "pawarshivam826@gmail.com"
                GITHUB_USERNAME = "shivampawar777"
                GITHUB_REPO = "streamlit_application-demo"
            }
            steps {
                withCredentials([string(credentialsId: "gitHub", variable: "GITHUB_TOKEN"),
                usernamePassword(credentialsId: "dockerhub",
                usernameVariable: "DOCKER_USER",
                passwordVariable: "DOCKER_PASS"
                )]) {
		            sh """
                        git config user.email "${env.GITHUB_EMAIL}"
                        git config user.name "${env.GITHUB_USERNAME}"

                        # update the deployment file: replace image tag
                        sed -i "s|${DOCKER_USER}/streamlit-app:.*|${DOCKER_USER}/streamlit-app:${env.IMAGE_TAG}|g" Deployment-Manifest/deployment.yml
                        
                        git add Deployment-Manifest/deployment.yml
                        git commit -m "Updated deployment image to version ${env.IMAGE_TAG}"
                        git push https://${GITHUB_TOKEN}@github.com/${GITHUB_USERNAME}/${GITHUB_REPO} HEAD:master
                    """        
                }
            }
        }
    }
}