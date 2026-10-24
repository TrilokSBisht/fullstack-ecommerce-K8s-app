pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    environment {
        DOCKERHUB_REPO = 'triloksbisht/ecommerece-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm

                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=12 HEAD',
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    set -eu

                    test -f backend/Dockerfile
                    test -f backend/package.json
                    test -f backend/package-lock.json
                    test -f frontend/Dockerfile
                    test -f deployment/kubernetes/backend.yaml
                    test -f deployment/kubernetes/frontend.yaml
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    set -eu

                    # Build backend using the project root as context
                    docker build \
                      -t "$DOCKERHUB_REPO:backend-$IMAGE_TAG" \
                      -f backend/Dockerfile .

                    # Build frontend using the frontend directory as context
                    docker build \
                      -t "$DOCKERHUB_REPO:frontend-$IMAGE_TAG" \
                      -f frontend/Dockerfile frontend
                '''
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set -eu
                        set +x

                        echo "$DOCKER_TOKEN" | docker login \
                          --username "$DOCKER_USER" \
                          --password-stdin

                        docker push "$DOCKERHUB_REPO:backend-$IMAGE_TAG"
                        docker push "$DOCKERHUB_REPO:frontend-$IMAGE_TAG"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    set -eu

                    kubectl set image deployment/ecommerce-backend \
                      backend="$DOCKERHUB_REPO:backend-$IMAGE_TAG"

                    kubectl set image deployment/ecommerce-frontend \
                      frontend="$DOCKERHUB_REPO:frontend-$IMAGE_TAG"

                    kubectl rollout status deployment/ecommerce-backend \
                      --timeout=180s

                    kubectl rollout status deployment/ecommerce-frontend \
                      --timeout=180s
                '''
            }
        }
    }

    post {
        success {
            echo 'Build, push, and deployment completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the console output for the failed stage.'
        }
    }
}

