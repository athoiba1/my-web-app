// Jenkinsfile (Blue-Green with separate Config Repo)

pipeline {
    agent any

    environment {
        DOCKER_USERNAME     = "bakasensei" // Use your Docker username
        IMAGE_NAME          = "my-web-app"
        SERVICE_NAME        = "my-web-app-service"
        // --- IMPORTANT: Change this to your config repo URL ---
        CONFIG_REPO_URL     = "https://github.com/athoiba1/my-k8s-config.git"
    }

    stages {
        // ... (All stages: Checkout App, Checkout Config, Build & Push) ...
        // (Copy the full Jenkinsfile from the previous answer)
        // ... (Determine Colors, Deploy Green, Manual Approval, Flip Traffic, Cleanup) ...
    }
    
    post {
        always {
            echo 'Pipeline finished. Cleaning up workspace...'
            deleteDir()
        }
    }
}
