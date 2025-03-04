pipeline {
    agent any

    environment {
        IMAGE_NAME = 'neamulkabiremon/jenkins-flask-app'
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
    }

    stages {
        stage('Setup') {
            steps {
                sh """
                    echo "🔧 Setting up environment..."

                    # Ensure Python dependencies are installed
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install pytest

                    # Ensure Docker is installed
                    echo "🔍 Checking if Docker is installed..."
                    if ! command -v docker &> /dev/null; then
                        echo "⚠️ Docker not found. Installing..."
                        sudo apt update -y
                        sudo apt install -y docker.io
                        sudo systemctl enable docker
                        sudo systemctl start docker
                        sudo usermod -aG docker jenkins
                        echo "✅ Docker installed successfully."
                    else
                        echo "✅ Docker is already installed."
                    fi

                    echo "✅ Setup completed."
                """
            }
        }

        stage('Test') {
            steps {
                sh """
                    echo "🧪 Running Tests..."
                    python3 -m pytest
                    echo "✅ Tests completed successfully."
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                        echo "🔑 Logging into Docker Hub..."
                        echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin
                        echo "✅ Docker login successful."
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    echo "🐳 Building Docker Image..."
                    docker build -t ${IMAGE_TAG} .
                    echo "✅ Docker image built successfully."
                    docker image ls
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    echo "📤 Pushing Docker Image..."
                    docker push ${IMAGE_TAG}
                    echo "✅ Docker image pushed successfully."
                """
            }
        }
    }

    post {
        failure {
            echo "❌ Build failed. Check the logs for more details."
        }
        success {
            echo "🎉 Build and deployment completed successfully!"
        }
    }
}
