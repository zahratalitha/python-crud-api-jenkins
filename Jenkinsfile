
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code berhasil di-checkout dari GitHub.'
            }
        }

        stage('Syntax Check') {
            steps {
                sh 'python -m py_compile app.py'
                echo 'Python syntax check berhasil.'
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t python-crud-api .'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing Backend API...'
                sh '''
                    docker run -d --name api-test -p 5001:5000 python-crud-api
                    sleep 3

                    python -c "import urllib.request; r=urllib.request.urlopen('http://localhost:5001/health'); assert r.status == 200"

                    python -c "import json, urllib.request, urllib.error; req=urllib.request.Request('http://localhost:5001/messages', data=json.dumps({'message':'Test Jenkins'}).encode(), headers={'Content-Type':'application/json'}, method='POST'); r=urllib.request.urlopen(req); assert r.status == 201"

                    python -c "import urllib.request, urllib.error; req=urllib.request.Request('http://localhost:5001/messages', data=b'{\\"message\\":\\"\\"}', headers={'Content-Type':'application/json'}, method='POST'); exec('try:\\n urllib.request.urlopen(req)\\n raise AssertionError(\\'Expected HTTP 400\\')\\nexcept urllib.error.HTTPError as e:\\n assert e.code == 400')"

                    docker stop api-test
                    docker rm api-test
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Backend API with persistent volume...'
                sh '''
                    docker stop python-crud-container || true
                    docker rm python-crud-container || true
                    docker volume create backend_data
                    docker run -d --name python-crud-container \
                        -p 5000:5000 \
                        -v backend_data:/var/lib/backend_data \
                        python-crud-api
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking deployed API...'
                sh '''
                    sleep 3
                    python -c "import urllib.request; r=urllib.request.urlopen('http://localhost:5000/health'); assert r.status == 200"
                    docker inspect --format='{{.State.Health.Status}}' python-crud-container
                '''
            }
        }
    }
}
