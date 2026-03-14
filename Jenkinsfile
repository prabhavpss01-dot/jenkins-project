pipeline {
    agent any

    triggers {
        githubPush()   // Webhook trigger
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prabhavpss01-dot/jenkins-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t ci-cd-sonarqube-docker .'
            }
        }

        stage('Test') {
            steps {
                bat 'docker run --rm ci-cd-sonarqube-docker pytest tests/test_app.py'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                   withSonarQubeEnv('SonarQubeServer') {
                     script {
                        def scannerHome = tool 'SonarScanner'
                        bat "${scannerHome}\\bin\\sonar-scanner -Dsonar.projectKey=jenkins-project -Dsonar.sources=app -Dsonar.tests=tests"
                            }
                    }
            }
    }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat 'docker login -u %USER% -p %PASS%'
                    bat 'docker tag ci-cd-sonarqube-docker %USER%/ci-cd-sonarqube-docker:latest'
                    bat 'docker push %USER%/ci-cd-sonarqube-docker:latest'
                }
            }
        }

        stage('Deploy') {
            steps {
                bat 'docker run -d -p 5000:8080 ci-cd-sonarqube-docker'
            }
        }
    }
}
