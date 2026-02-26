pipeline {
    agent any
    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                # Remove existing image if it exists to ensure a fresh build
                docker rmi -f backend-app || true
                
                # Build using the relative path to the backend folder
                docker build -t backend-app ./backend
                '''
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create the network if it doesn't exist
                docker network create app-network || true
                
                # Remove old containers to prevent name conflicts
                docker rm -f backend1 backend2 || true
                
                # Start two instances of the backend
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Remove old Nginx container
                docker rm -f nginx-lb || true
                
                # Start fresh Nginx container on port 80
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx
                
                # Give the container a few seconds to initialize
                sleep 5
                
                # Copy the config from your repo's nginx folder to the container
                docker cp ./nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                
                # Reload Nginx to apply the new configuration
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}