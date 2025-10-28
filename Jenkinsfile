// Jenkinsfile (Blue-Green with separate Config Repo)

pipeline {
    agent any

    environment {
        DOCKER_USERNAME     = "bakasensei" // Your Docker username
        IMAGE_NAME          = "my-web-app"
        SERVICE_NAME        = "my-web-app-service"
        // --- Make sure this is your config repo URL ---
        CONFIG_REPO_URL     = "https://github.com/athoiba1/my-k8s-config.git"
    }

    stages {
        // -----------------------------------------------------------------
        // STAGE 1: Checkout App Code
        // -----------------------------------------------------------------
        stage('Checkout App') {
            steps {
                checkout scm
            }
        }
        
        // -----------------------------------------------------------------
        // STAGE 2: Checkout Config Repo
        // -----------------------------------------------------------------
        stage('Checkout Config') {
            steps {
                echo "Checking out Kubernetes config from ${CONFIG_REPO_URL}"
                dir('config-repo') {
                    git url: CONFIG_REPO_URL, branch: 'main'
                }
            }
        }

        // -----------------------------------------------------------------
        // STAGE 3: Build & Push Image
        // -----------------------------------------------------------------
        stage('Build & Push Image') {
            steps {
                script {
                    def imageWithTag = "${DOCKER_USERNAME}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    echo "Building & Pushing image: ${imageWithTag}"
                    
                    docker.build(imageWithTag)
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-creds') {
                        docker.image(imageWithTag).push()
                    }
                }
            }
        }

        // -----------------------------------------------------------------
        // STAGE 4: Determine Deployment Colors (FIXED)
        // -----------------------------------------------------------------
        stage('Determine Colors') {
            steps {
                script {
                    echo "Checking which color is currently live..."
                    
                    // --- FIX: Use double quotes and .replace() to clean the output ---
                    env.LIVE_COLOR = bat(
                        returnStdout: true,
                        script: "@kubectl get service ${SERVICE_NAME} -o=jsonpath='{.spec.selector.color}'"
                    ).trim().replace("'", "") // <-- This strips the quotes

                    if (env.LIVE_COLOR == "blue") {
                        env.NEW_COLOR = "green"
                    } else {
                        env.NEW_COLOR = "blue"
                    }
                    
                    echo "Live color is: ${env.LIVE_COLOR}. New color will be: ${env.NEW_COLOR}."
                }
            }
        }

        // -----------------------------------------------------------------
        // STAGE 5: Deploy "Green" (The New Version)
        // -----------------------------------------------------------------
        stage('Deploy New Version (Green)') {
            steps {
                script {
                    def newAppName = "my-web-app-${env.NEW_COLOR}"
                    def imageWithTag = "${DOCKER_USERNAME}/${IMAGE_NAME}:${env.BUILD_NUMBER}"

                    echo "Deploying new version to ${newAppName} with image ${imageWithTag}"

                    def deploymentText = readFile 'config-repo/deployment-template.yaml'
                    
                    deploymentText = deploymentText.replace('${APP_NAME}', newAppName)
                                                .replace('${COLOR}', env.NEW_COLOR)
                                                .replace('${IMAGE_WITH_TAG}', imageWithTag)
                    
                    writeFile file: 'new-deployment.yaml', text: deploymentText
                    
                    withKubeConfig([credentialsId: 'kubeconfig-creds']) {
                        bat "kubectl apply -f new-deployment.yaml"
                    }
                }
            }
        }
        
        // -----------------------------------------------------------------
        // STAGE 6: Manual Approval (Test "Green" Environment)
        // -----------------------------------------------------------------
        stage('Manual Approval') {
            steps {
                echo "New version '${env.NEW_COLOR}' is deployed but not live."
                echo "Test it by running: kubectl port-forward deployment/my-web-app-${env.NEW_COLOR} 9090:8080"
                
                input "Ready to switch live traffic to ${env.NEW_COLOR}?"
            }
        }
        
        // -----------------------------------------------------------------
        // STAGE 7: Flip Live Traffic
        // -----------------------------------------------------------------
        stage('Flip Live Traffic') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig-creds']) {
                        echo "Flipping switch! Pointing ${SERVICE_NAME} selector to color: ${env.NEW_COLOR}"

                        def patchContent = "{\"spec\":{\"selector\":{\"app\":\"my-web-app\",\"color\":\"${env.NEW_COLOR}\"}}}"
                        writeFile file: 'patch.json', text: patchContent
                        
                        bat "kubectl patch service ${SERVICE_NAME} --patch-file patch.json"
                        
                        echo "Success! ${env.NEW_COLOR} is now live."
                    }
                }
            }
        }
        
        // -----------------------------------------------------------------
        // STAGE 8: Cleanup Old Version (Blue)
        // -----------------------------------------------------------------
        stage('Cleanup Old Version') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-creds']) {
                    echo "Cleaning up old deployment: my-web-app-${env.LIVE_COLOR}"
                    bat "kubectl delete deployment my-web-app-${env.LIVE_COLOR}"
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline finished. Cleaning up workspace...'
            deleteDir()
        }
    }
}
