pipeline {
    agent any

    tools {
    nodejs 'node20'
  }

    environment {
        APP_NAME   = "node-app"
        IMAGE_TAG  = "v1"
        REGISTRY   = "13.204.233.144:8083"   // Your Nexus Docker repo
        SONAR_HOST = "http://13.204.233.144:9000"
        TRIVY_PATH = "/usr/local/bin/trivy" // Or container alias
    }

    stages {

         stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/gosulaeswar2498/DevSecops_Project.git'
                }
        } 
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv('My-sonarqube') {
                    sh """
                       ${SCANNER_HOME}/bin/sonar-scanner \
                       -Dsonar.projectKey=node-app \
                       -Dsonar.sources=. \
                       -Dsonar.host.url=${SONAR_HOST}
                    """
                }
            }
        }

        stage('Wait for Sonar Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Node.js App') {
            steps {
                sh '''
                    if npm run | grep -q build; then
                       npm run build
                    else
                       echo "No build step found"
                    fi
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${APP_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Trivy Scan') {
            steps {
               sh """
                   docker run --rm \
                   -v /var/run/docker.sock:/var/run/docker.sock \
                   aquasec/trivy:latest image node-app:v1 || true //set to false if pipeline want to get fails if docker image is not compliant. 
                  """
    }
}

        stage('Docker Push to Nexus') {
            steps {
                sh """
                    docker tag ${APP_NAME}:${IMAGE_TAG} ${REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                    docker push ${REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                """
            }
        }

    }
}
