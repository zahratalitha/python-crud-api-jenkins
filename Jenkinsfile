
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
                echo 'Checking Python syntax...'
                sh 'python3 -m py_compile app.py'
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
                    docker rm -f api-test || true

                    docker run -d \
                        --name api-test \
                        -p 5001:5000 \
                        python-crud-api

                    sleep 5

                    python3 -c "
import urllib.request
r = urllib.request.urlopen('http://localhost:5001/health')
assert r.status == 200
print('Health test: PASSED')
"

                    python3 -c "
import json
import urllib.request

data = json.dumps({'message': 'Test Jenkins'}).encode()

req = urllib.request.Request(
    'http://localhost:5001/messages',
    data=data,
    headers={'Content-Type': 'application/json'},
    method='POST'
)

r = urllib.request.urlopen(req)
result = json.loads(r.read())

assert r.status == 201
assert 'created_at' in result['message']
print('POST valid test: PASSED')
"

                    python3 -c "
import urllib.request
import urllib.error

data = b'{\\"message\\":\\"\\"}'

req = urllib.request.Request(
    'http://localhost:5001/messages',
    data=data,
    headers={'Content-Type': 'application/json'},
    method='POST'
)

try:
    urllib.request.urlopen(req)
    raise AssertionError('Expected HTTP 400')
except urllib.error.HTTPError as e:
    assert e.code == 400
    print('POST empty test: PASSED')
"

                    docker rm -f api-test
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Backend API with persistent volume...'

                sh '''
                    docker rm -f python-crud-container || true

                    docker volume create backend_data

                    docker run -d \
                        --name python-crud-container \
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
                    sleep 5

                    python3 -c "
import urllib.request
r = urllib.request.urlopen('http://localhost:5000/health')
assert r.status == 200
print('Deployment health check: PASSED')
"

                    docker inspect \
                        --format='{{.State.Health.Status}}' \
                        python-crud-container
                '''
            }
        }
    }
}
