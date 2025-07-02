pipeline {
    agent any

    environment {
        DOCKER_USER = credentials('DOCKER_USER')
        DOCKER_PASS = credentials('DOCKER_PASS')
        SONAR_TOKEN = credentials('SONAR_TOKEN')
    }

    stages {
        stage('Cloner le repo') {
            steps {
                git 'https://github.com/sailorKei/cicd-tp'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'echo "Build et tests ici (ex: npm install && npm test)"'
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner -Dsonar.projectKey=tp-cicd -Dsonar.sources=. -Dsonar.login=$SONAR_TOKEN'
                }
            }
        }

        stage('Build Docker image') {
            steps {
                sh 'docker build -t keichan02/tp-cicd-app:latest .'
            }
        }

        stage('Push DockerHub') {
            steps {
                sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push keichan02/tp-cicd-app:latest
                '''
            }
        }
    }
}
