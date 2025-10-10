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

        stage('Docker Build & Push') {
            steps {
                script {
                    // DockerHub credentials
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhuhpass',
                        usernameVariable: 'docker_user',
                        passwordVariable: 'docker_password'
                    )]) {
                        sh '''
                            echo "$docker_password" | docker login -u "$docker_user" --password-stdin
                            docker build -t $IMAGE_NAME .
                            docker tag $IMAGE_NAME $DOCKER_REPO:latest
                            docker push $DOCKER_REPO:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // AWS credentials for EKS kubeconfig
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                        sh '''
                            export AWS_DEFAULT_REGION=ap-south-1
                            kubectl apply -f /var/lib/jenkins/workspace/job/deployment.yml
                            kubectl apply -f /var/lib/jenkins/workspace/job/service.yml
                            kubectl get svc
                        '''
                    }
                }
            }
        }
    }
}
