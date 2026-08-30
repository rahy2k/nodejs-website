pipeline {
    agent any
    tools {
        // Fix: Use 'type:' explicitly instead of a raw string
        type type: 'hudson.plugins.sonar.SonarRunnerInstallation', name: 'Sonar-Scanner'
    }
    stages {
        stage('Sonar') {
            steps {
                echo 'Running analysis...'
            }
        }
    }
     
stage('SonarQube Analysis') {
            steps {
                script {
                    // Fetch the installation path directly
                    def sonarHome = tool name: 'Sonar-Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    
                    // Add it to the PATH and execute
                    withEnv(["PATH+SONAR=${sonarHome}/bin"]) {
                        sh 'sonar-scanner'
                    }
                }
            }
        }
}
