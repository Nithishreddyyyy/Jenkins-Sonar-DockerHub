pipeline {
    agent any

    tools {
        sonarQube 'SonarScanner'
    }

    environment {
        DOCKER_IMAGE = "YOUR_DOCKER_USERNAME/ml-quality-pipeline:latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/YOUR_USERNAME/ml-quality-pipeline.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m pip install -r requirements.txt
                '''
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh '''
                    sonar-scanner
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
    }
}
