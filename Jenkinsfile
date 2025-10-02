pipeline {
    agent {
        docker {
            image 'shivam139/jenkins-agent-ubuntu:v2'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    stages {
        stage('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/shivampawar777/streamlit_application.git' 
                sh 'echo passed...'
            }
        }
        stage('Build & push docker image') {
            steps {
                echo 'Building the docker image And then pushing image to the dockerHub'
                withCredentials([usernamePassword(credentialsId: "dockerhub", usernameVariable: "dockerhubUser", passwordVariable: "dockerhubPass")]) {
                    sh """
                        cd Python-app && docker build -t ${env.dockerhubUser}/streamlit_application:${BUILD_NUMBER} .
                        docker login -u ${env.dockerhubUser} -p ${env.dockerhubPass}
                        docker push ${env.dockerhubUser}/streamlit_application:${BUILD_NUMBER}
                    """
                }
            }
        }
        stage('Update deployment manifest') {
            environment {
                GITHUB_EMAIL = "pawarshivam826@gmail.com"
                GITHUB_USERNAME = "shivampawar777"
                GITHUB_REPO = "streamlit_application-demo"
		IMAGE = "v${BUILD_NUMBER}"
            }
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerhub", usernameVariable: "dockerhubUser"), string(credentialsId: "gitHub", variable: "GITHUB_TOKEN")]) {
		    sh """
                        git config user.email "${GITHUB_EMAIL}"
                        git config user.name "${GITHUB_USERNAME}"
                        sed -i "s|${env.dockerhubUser}/streamlit-app:.*|${env.dockerhubUser}/streamlit-app:${IMAGE}|g" Deployment-Manifest/deployment.yml
                        git add Deployment-Manifest/deployment.yml
                        git commit -m "Updated deployment image to version ${IMAGE}"
                        git push https://${env.GITHUB_TOKEN}@github.com/${GITHUB_USERNAME}/${GITHUB_REPO} HEAD:main
                    """        
                }
            }
        }
    }
}
