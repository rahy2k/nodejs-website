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
     
}
