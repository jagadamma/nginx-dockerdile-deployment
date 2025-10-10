pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        IMAGE_NAME = 'myimg'
        DOCKER_REPO = 'jagadamma/nginx'
        GIT_REPO = 'https://github.com/jagadamma/nginx-dockerdile-deployment.git'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git "${GIT_REPO}"
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                script {
                    // Inject DockerHub username/password from Jenkins credentials
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhuhpass', 
                        usernameVariable: 'docker_user', 
                        passwordVariable: 'docker_password'
                    )]) {
                        sh '''
                            echo "Logging in to DockerHub..."
                            echo "$docker_password" | docker login -u "$docker_user" --password-stdin
                            docker tag $IMAGE_NAME:latest $DOCKER_REPO:latest
                            docker push $DOCKER_REPO:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Run all commands in a single shell
                sh '''
                    cd /var/lib/jenkins/workspace/job
                    kubectl apply -f deployment.yml 
                    kubectl apply -f service.yml 
                    kubectl get svc
                '''
            }
        }
    }
}
