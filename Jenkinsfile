pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-website"
        CONTAINER_NAME = "my-website-container"
        APP_PORT = "3000"
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo '📥 Cloning repository...'
                git branch: 'main',
                    url: 'https://github.com/justicesecops-10/git.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '📦 Installing Node.js dependencies...'
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo '🧪 Running tests...'
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                echo '🏗️ Running build step...'
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Deploy Container') {
            steps {
                echo '🚀 Deploying container...'
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm   ${CONTAINER_NAME} || true

                    OLD=\$(docker ps -q --filter "publish=${APP_PORT}")
                    if [ ! -z "\$OLD" ]; then
                        docker stop \$OLD || true
                        docker rm \$OLD || true
                    fi

                    sleep 2

                    docker run -d \\
                        --name ${CONTAINER_NAME} \\
                        -p 0.0.0.0:${APP_PORT}:${APP_PORT} \\
                        --restart unless-stopped \\
                        ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Health Check') {
            steps {
                echo '❤️ Checking app health...'
                sh 'sleep 3'
                sh "curl -f http://localhost:${APP_PORT} || exit 1"
                echo '✅ App is live!'
            }
        }

        stage('Cleanup Old Images') {
            steps {
                echo '🧹 Cleaning up old Docker images...'
                sh 'docker image prune -f || true'
            }
        }
    }

    post {
        success {
            echo "✅ ============================================"
            echo "✅ PIPELINE SUCCESS!"
            echo "✅ App running at http://13.203.197.53:${APP_PORT}"
            echo "✅ ============================================"
        }
        failure {
            echo "❌ PIPELINE FAILED — Check logs above"
            sh """
                docker ps -a || true
                docker logs ${CONTAINER_NAME} || true
            """
        }
        always {
            echo "Pipeline finished."
            sh "docker ps"
        }
    }
}
