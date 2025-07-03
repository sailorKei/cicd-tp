pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        DOCKER_CRED = 'dockerhub'
        SONAR_TOKEN = credentials('sonar-token')
        SONAR_URL = 'http://13.61.190.120:9000'
        NEXUS_URL = 'http://13.61.190.120:8081/repository/maven-snapshots/'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package -B'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    withSonarQubeEnv('MySonar') {
                        sh "mvn sonar:sonar -Dsonar.host.url=${SONAR_URL} -Dsonar.login=${SONAR_TOKEN}"
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def imageName = "keichan02/cicd-tp"
                    sh "docker build -t ${imageName} ."
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', passwordVariable: 'NEXUS_PASS', usernameVariable: 'NEXUS_USER')]) {
                    sh 'mvn deploy -DaltDeploymentRepository=nexus::default::' +
                       "${NEXUS_URL} -Dnexus.user=${NEXUS_USER} -Dnexus.password=${NEXUS_PASS}"
                }
            }
        }
    }
}
