
pipeline {
    agent any

    environment {
        IMAGE_NAME = "docker.io/kavitamil13/spring-cloud-gateway"
        K8S_NAMESPACE = "default"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/kavitamli13/spring-cloud-gateway.git',
                    branch: 'main'
            }
        }

        stage('Build Java App') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t $IMAGE_NAME:latest .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }


        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    string(credentialsId: 'k8s-token', variable: 'K8S_TOKEN'),
                    string(credentialsId: 'k8s-api', variable: 'K8S_API')
                ]) {
                    sh """
                    kubectl config set-cluster k8s --server=$K8S_API --insecure-skip-tls-verify=true
                    kubectl config set-credentials jenkins --token=$K8S_TOKEN
                    kubectl config set-context jenkins --cluster=k8s --user=jenkins --namespace=${K8S_NAMESPACE}
                    kubectl config use-context jenkins

                    sed -i 's|DOCKER_IMAGE|${IMAGE_NAME}:${BUILD_NUMBER}|g' k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }
    }
}
