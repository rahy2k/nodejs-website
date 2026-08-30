pipeline {
    agent any

    tools {
        // References the scanner tool from Step 5
        type 'hudson.plugins.sonar.SonarRunnerInstallation', name: 'Sonar-Scanner'
    }

    stages {
        stage('Clone Source') {
            steps {
                git branch: 'main', url: 'https://github.com'
            }
        }

        stage('SonarQube Cloud Analysis') {
            steps {
                // Must match the system name "SonarCloud" used in Step 4
                withSonarQubeEnv('SonarCloud') {
                    sh 'sonar-scanner \
                        -Dsonar.organization=your-sonarcloud-organization-key \
                        -Dsonar.projectKey=your-unique-project-key \
                        -Dsonar.projectName="Your Project Name" \
                        -Dsonar.sources=.'
                }
            }
        }
        
        stage("Quality Gate Check") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Pauses build execution until the Cloud checks pass
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
