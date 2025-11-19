pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'ahmedhoucine0/mon-app'
        HELM_CHART_PATH = './mon-app-helm'
    }
    stages {
        stage('Cloner le dépôt') {
            steps {
                git 'https://github.com/ahmedhoucine/tp3-devops.git'
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
        stage('Déployer avec Helm') {
            steps {
                script {
                    sh '''
                        helm upgrade --install mon-app $HELM_CHART_PATH \
                        --set image.repository=$DOCKER_IMAGE \
                        --set image.tag=latest
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline Helm terminé'
        }
    }
}