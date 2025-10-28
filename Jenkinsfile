pipeline {
    agent any
    environment {
        DOCKER_USERNAME = "bakasensei" 
        IMAGE_NAME = "my-web-app"
        K8S_DEPLOYMENT_NAME = "my-web-app-deployment" 
    }
    stages {
        stage('Checkout') {
            steps {
                // This is the fix. It uses the branch from the job config.
                checkout scm 
            }
        }