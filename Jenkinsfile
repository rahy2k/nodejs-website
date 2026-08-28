pipeline {
    agent any

    environment {
        // Name configured in Jenkins:
        // Manage Jenkins → System → SonarQube servers
        SONARQUBE = 'SonarQube'

        // SonarQube project key
        SONAR_PROJECT_KEY = 'nodejs-website'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm ci'
            }
        }

        stage('Code Test') {
            steps {
                echo 'Running code tests...'
                sh 'npm test'
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo 'Running SonarQube analysis...'

                withSonarQubeEnv("${SONARQUBE}") {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.sources=. \
                          -Dsonar.exclusions=node_modules/**,coverage/**
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate...'

                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Apache') {
            steps {
                echo 'Deploying application...'

                sh '''
                    sudo -n rsync -av --delete \
                      --exclude=".git" \
                      --exclude=".github" \
                      --exclude="node_modules" \
                      ./ /var/www/html/

                    sudo -n chown -R www-data:www-data /var/www/html

                    sudo -n systemctl restart apache2
                '''
            }
        }
    }

    post {
        success {
            echo 'Build, SonarQube scan and deployment completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}


