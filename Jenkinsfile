pipeline {
agent any
environment {
IMAGE = "onepage-app:${env.BUILD_NUMBER}"
CONTAINER_NAME = "bee-onepage-cv"
HOST_PORT = "5000"
}

stages {
stage('Checkout Code') {
steps {
git branch: 'master', url: 'https://github.com/fulldrill/onepagecv.git'
}
}

stage('Build Docker Image') {
steps {
// use .\app for Windows paths when running inside bat
bat "docker build -t %IMAGE% .\\app"
}
}

stage('Run Container') {
steps {
bat '''
REM remove existing container if present; ignore error if not present
docker rm -f %CONTAINER_NAME% || exit /b 0

REM run container detached, mapping HOST_PORT->80
docker run -d -p %HOST_PORT%:80 --name %CONTAINER_NAME% %IMAGE%
'''
}
}

stage('Nagios Health Check - HTTP') {
steps {
// fail the step (and pipeline) if HTTP check fails
bat 'curl -f http://localhost:%HOST_PORT%/ || exit /b 1'
}
}

stage('Nagios Health Check - Ping') {
steps {
bat 'ping -n 2 localhost || exit /b 1'
}
}

stage('Post-Deployment Report') {
steps {
echo "Deployment complete. Nagios checks passed. Application is healthy."
}
}
}

post {
failure {
echo " Deployment failed or Nagios checks failed!"
}
}
}
