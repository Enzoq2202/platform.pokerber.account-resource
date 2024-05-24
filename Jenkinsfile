pipeline {
    agent any
    stages {
        stage('Build Account') {
            steps {
                build job: 'store.account', wait: true
            }
        }
        stage('Build') { 
            steps {
                sh 'mvn clean package'
            }
        }      
        stage('Build Image') {
            steps {
                script {
                    account = docker.build("enzoq2202/account:${env.BUILD_ID}", "-f Dockerfile .")
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        account.push("${env.BUILD_ID}")
                        account.push("latest")
                    }
                }
            }
        }
        stage('Deploy on k8s') {
            steps {
                withCredentials([ string(credentialsId: 'minikube_credentials', variable: 'api_token') ]) {
                    sh 'kubectl --token $api_token --server https://host.docker.internal:62598  --insecure-skip-tls-verify=true apply -f ./k8s/deployment.yaml'
                    sh 'kubectl --token $api_token --server https://host.docker.internal:62598  --insecure-skip-tls-verify=true apply -f ./k8s/service.yaml'
                }
            }
        }
    }
}