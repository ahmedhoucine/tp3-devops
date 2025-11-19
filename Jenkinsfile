pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'ahmedhoucine0/mon-app'
    }
    stages {
         stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }
        stage('Cloner le dépôt') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ahmedhoucine/tp3-devops.git'
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
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }
        
        stage('Pousser l\'image Docker') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', 
                    passwordVariable: 'DOCKER_PASSWORD', 
                    usernameVariable: 'DOCKER_USERNAME'
                )]) {
                    sh '''
                        echo "Login à Docker Hub..."
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        echo "Push de l'image..."
                        docker push ''' + DOCKER_IMAGE + '''
                    '''
                }
            }
        }
        
        stage('Déployer sur Kubernetes') {
    steps {
        sh '''
            echo "=== Installation des outils ==="
            # Installation kubectl
            curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
            chmod +x kubectl
            mv kubectl /usr/local/bin/
            
            # Installation Minikube
            curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
            chmod +x minikube-linux-amd64
            mv minikube-linux-amd64 /usr/local/bin/minikube
            
            echo "=== Démarrage de Minikube dans Jenkins ==="
            minikube start --driver=docker --force --wait=true --wait-timeout=5m
            
            echo "=== Vérification ==="
            minikube status
            kubectl cluster-info
            kubectl get nodes
            
            echo "=== Déploiement ==="
            kubectl apply -f deployment.yaml
            kubectl apply -f service.yaml
            
            echo "=== Vérification ==="
            kubectl get deployments,services,pods
        '''
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