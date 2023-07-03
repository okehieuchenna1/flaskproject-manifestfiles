pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REPO = "okehieuchenna/flask-project"
        CONTAINER_NAME = "flask-project"
        DOCKHUB_CREDENTIALS = credentials('dockerhub-login')
        DOCKERHUB_TOKEN = credentials('docker-login-token')
    }

    stages {
        stage('Clone GitHub Repo') {
            steps {
                echo 'Cloning SCM Repo'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/okehieuchenna1/flaskproject']])
            }
        }
        stage('Build') {
            steps {
                echo 'Building docker image..'
                sh 'docker build -t $DOCKER_HUB_REPO:${BUILD_NUMBER} .'
            }
        }
        stage('Push Build to dockerhub') {
            steps {
                echo 'Pushing docker image..'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u okehieuchenna --password $DOCKERHUB_TOKEN'
                sh 'docker push $DOCKER_HUB_REPO:${BUILD_NUMBER}'
            }
        }
        stage('Delete Build on Local repo') {
            steps {
                echo 'Deleting docker image..'
                sh 'docker rmi -f $DOCKER_HUB_REPO:${BUILD_NUMBER}'
            }
        }
        stage('Updating K8s deployment file') {
            steps {
                echo 'Executing update on k8s deployment file..'
                script{
                    sh """
                    cat deployment.yaml
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yaml
                    cat deployment.yaml
                    """
                }
            }
        }
        stage('Pushing Changed deployment file to github') {
            steps {
                echo 'Pushing updated deployment to GitHub..'
            script{
                sh """
                  git config --global user.name "Uchenna Okehie"
                  git config --global user.email "okehieuchenna@gmail.com"
                  git add deployment.yaml
                  git commit -m "updating the deployment file to ${BUILD_NUMBER} build"
                  
 
                """
                withCredentials([usernamePassword(credentialsId: 'eddd0f91-ff01-4f7b-ad30-ddda1001fa51',
                usernameVariable: 'username',
                passwordVariable: 'password')]){
                  sh "git push https://$username:$password@github.com/okehieuchenna1/flaskproject-manifestfiles.git HEAD:main --force"
                 }         
                
                }
                
            }
        }
        stage('Deploying file on Kubernetes Cluster') {
            steps {
                echo 'Deploying on K8s Cluster..'
                script{
                    sh """
                        kubectl apply -f https://raw.githubusercontent.com/okehieuchenna1/flaskproject-manifestfiles/main/deployment.yaml
                        kubectl apply -f https://raw.githubusercontent.com/okehieuchenna1/flaskproject-manifestfiles/main/service.yaml
                    """
                }
            }

        }
        
    }
    
}
