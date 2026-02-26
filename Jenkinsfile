pipeline {
    agent any

    parameters {
        choice(
            name: 'Backend_Count',
            choices: ['1','2'],
            description: 'Number of backend containers to deploy'
        )
    }

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker network create app-network || true
                docker rm -f backend1 backend2 || true

                if [ "$Backend_Count" = "1" ]; then
                    docker run -d --name backend1 --network app-network backend-app
                else
                    docker run -d --name backend1 --network app-network backend-app
                    docker run -d --name backend2 --network app-network backend-app
                fi
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
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
