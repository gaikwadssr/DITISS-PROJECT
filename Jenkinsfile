pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('WebServer-Sonar') // Fetch the SonarQube token from Jenkins credentials
        DOCKERHUB_CREDENTIALS = credentials('DOCKERHUB')
        
    }

    stages {
        stage('SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/SnehaKalsait/Project-Sunbeam.git'
            }
        }
        
         stage('SonarQube Analysis') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'WebServer-Sonar', variable: 'SONAR_TOKEN')]) {
                        sh """
                        /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarScanner/bin/sonar-scanner \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.projectKey=WebServer \
                            -Dsonar.sources=./index.html \
                            -Dsonar.token=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t snehakalsait/webserver .'
            }
        }

        stage('Dockerhub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push to Dockerhub') {
            steps {
                sh 'docker push snehakalsait/webserver'
            }
        }
        
        stage('Run SSH Command') {
            steps {
                sshagent(['Project']) {
                    sh """
                ssh -o StrictHostKeyChecking=no ubuntu@3.109.122.212 "
                    echo 'Hello from Jenkins' && hostname
                    docker stop website1
                    docker pull snehakalsait/webserver:latest
                    docker rm website1
                    docker run -d --name website1 -p 80:80 snehakalsait/webserver:latest
                "
                """
                }
            }
        }
    }
}
