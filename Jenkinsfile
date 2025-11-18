pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'votre-username/mon-app'
        KUBECONFIG = credentials('kubeconfig')
    }
    stages {
        stage('Cloner le dépôt') {
            steps {
                git 'https://github.com/votre-username/mon-app.git'
            }
        }
        stage('Construire l\'image Docker') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }
        stage('Pousser l\'image Docker') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }
        stage('Déployer sur Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline terminé'
        }
        success {
            echo 'Déploiement réussi!'
        }
        failure {
            echo 'Échec du déploiement'
        }
    }
}