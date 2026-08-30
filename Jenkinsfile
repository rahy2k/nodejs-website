pipeline {
    agent any
    environment {
        // This automatically finds the tool named 'Sonar-Scanner'
        SONAR_RUNNER_HOME = tool 'Sonar-Scanner'
    }
    stages {
        stage('SonarQube Analysis') {
            steps {
                // Use the environment variable in your build step
                sh "${SONAR_RUNNER_HOME}/bin/sonar-scanner"
            }
        }
    }
}
