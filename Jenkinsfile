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
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }
        
        stage('Déployer sur Kubernetes') {
    steps {
        sh '''
            set -e
            echo "=== Création d'un cluster Kubernetes isolé dans Jenkins ==="
            
            # Installation kubectl
            curl -LO "https://dl.k8s.io/release/v1.28.0/bin/linux/amd64/kubectl"
            chmod +x kubectl
            sudo mv kubectl /usr/local/bin/
            
            # Installation Kind
            curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
            chmod +x ./kind
            sudo mv ./kind /usr/local/bin/kind
            
            echo "=== Création du cluster Kind ==="
            # Créer un fichier de configuration pour Kind
            cat > kind-config.yaml << EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
EOF
            
            kind create cluster --config kind-config.yaml --name jenkins-cluster --wait 2m
            
            echo "=== Vérification du cluster ==="
            kubectl cluster-info
            kubectl get nodes
            
            echo "=== Déploiement de l'application ==="
            kubectl apply -f deployment.yaml
            kubectl apply -f service.yaml
            
            echo "=== Vérification ==="
            kubectl get all
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