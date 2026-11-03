pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building Python CRUD API...'
                sh 'docker build -t python-crud-api .'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing Backend API...'
                sh 'echo "Pipeline test successful"'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Backend API...'
                sh '''
                    docker stop python-crud-container || true
                    docker rm python-crud-container || true
                    docker run -d --name python-crud-container -p 5000:5000 python-crud-api
                '''
            }
        }
    }
}