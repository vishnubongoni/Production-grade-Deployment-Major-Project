pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "viishnu24/multibranch-flask-app"
        GIT_USER = "viishnu24"
        GIT_EMAIL = "vishnubongoni99@gmail.com"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when { branch 'main' }
            steps {
                script {
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([usernamePassword(
                        credentialsId: "dockerhub-creds",
                        usernameVariable: "DOCKER_USER",
                        passwordVariable: "DOCKER_PASS"
                    )]) {
                       sh '''
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}    
                        '''
                    }
                }
            }
        }

        stage("Update k8s manifest") {
            when { branch 'main' }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                        set -e
                        git config user.name "$GIT_USER"
                        git config user.email "$GIT_EMAIL"

                        git fetch origin
                        git checkout main
                        git reset --hard origin/main

                       sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yaml

                        git add k8s/deployment.yml
                        git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/vishnubongoni/Production-grade-Deployment-Major-Project.git
                        """
                    }
                }
            }
        }
    }
}
