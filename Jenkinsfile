pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        SONAR_TOKEN = credentials('sonar-token')
        NEXUS_CREDENTIALS = credentials('nexus-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/sailorKei/cicd-tp.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    dockerImage = docker.build("keichan02/cicd-tp")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                sh """
                mvn deploy:deploy-file \
                    -DgroupId=com.example \
                    -DartifactId=cicd-tp \
                    -Dversion=1.0 \
                    -Dpackaging=jar \
                    -Dfile=target/*.jar \
                    -DrepositoryId=nexus \
                    -Durl=http://<TON_IP_PUBLIC>:8081/repository/maven-releases/ \
                    -Dusername=${NEXUS_CREDENTIALS_USR} \
                    -Dpassword=${NEXUS_CREDENTIALS_PSW}
                """
            }
        }
    }
}
