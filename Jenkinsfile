pipeline {
  agent any
  stages {
    stage('Checkout'){ steps{ git 'https://github.com/example/repo.git' } }
    stage('Install'){ steps{ sh 'npm install' } }
    stage('Build'){ steps{ sh 'npm run build || echo no-build' } }
    stage('Docker Build'){ steps{ sh 'docker build -t node-app:latest .' } }
  }
}
