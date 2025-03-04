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
                    echo "🔧 Setting up Python environment..."
                    
                    # Ensure virtual environment is created
                    python3 -m venv venv
                    source venv/bin/activate

                    # Upgrade pip
                    pip install --upgrade pip

                    # Install dependencies
                    pip install -r requirements.txt
                    pip install pytest

                    echo "✅ Setup completed."
                """
            }
        }

        stage('Test') {
            steps {
                sh """
                    echo "🧪 Running Tests..."
                    source venv/bin/activate
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
        always {
            echo "🛑 Cleaning up workspace..."
            sh "rm -rf venv"
            echo "✅ Cleanup completed."
        }
        failure {
            echo "❌ Build failed. Check the logs for more details."
        }
        success {
            echo "🎉 Build and deployment completed successfully!"
        }
    }
}
