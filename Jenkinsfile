pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'votre-username-docker/mon-app'
    }
    stages {
        stage('Cloner le dépôt') {
            steps {
                git 'https://github.com/ahmedhoucine/tp3-devops.git'
            }
        }
        
        stage('Vérifier les fichiers') {
            steps {
                sh '''
                    echo "Liste des fichiers dans le dépôt:"
                    ls -la
                    echo "Contenu du répertoire:"
                    find . -type f -name "*.yaml" -o -name "*.yml" -o -name "Dockerfile" | head -20
                '''
            }
        }
        
        stage('Construire l\'image Docker') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        
        stage('Pousser l\'image Docker') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds', 
                        passwordVariable: 'DOCKER_PASSWORD', 
                        usernameVariable: 'DOCKER_USERNAME'
                    )]) {
                        sh """
                            echo "Login à Docker Hub..."
                            docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                            echo "Push de l'image..."
                            docker push ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }
        
        stage('Déployer sur Kubernetes') {
            steps {
                script {
                    withCredentials([file(
                        credentialsId: 'kubeconfig', 
                        variable: 'KUBECONFIG_FILE'
                    )]) {
                        sh """
                            echo "Configuration de kubectl..."
                            export KUBECONFIG=${KUBECONFIG_FILE}
                            echo "Vérification du cluster..."
                            kubectl cluster-info
                            kubectl get nodes
                            echo "Déploiement de l'application..."
                            kubectl apply -f deployment.yaml
                            kubectl apply -f service.yaml
                            echo "Vérification du déploiement..."
                            kubectl get deployments
                            kubectl get services
                            kubectl get pods
                        """
                    }
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