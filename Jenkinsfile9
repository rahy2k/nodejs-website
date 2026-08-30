pipeline {
    agent any

    environment {
        APP_DIR = '/home/vboxuser/nodejs-website'
        BRANCH = 'main'
        SERVER = 'vboxuser@192.168.1.107'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Node.js Check') {
            steps {
                sh 'npm run check'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    sh 'mkdir -p reports'
                    try {
                        withSonarQubeEnv('SonarQube') {
                            sh '''#!/usr/bin/env bash
set +e
sonar-scanner \
    -Dsonar.projectKey=student-management-system \
    -Dsonar.projectName="Student Management System" \
    -Dsonar.sources=. \
    -Dsonar.tests=test \
    -Dsonar.exclusions=node_modules/**,output/**,test/** \
    2>&1 | tee reports/sonar-scan.log
SONAR_STATUS=${PIPESTATUS[0]}
set -e
exit $SONAR_STATUS
'''
                        }
                    } catch (err) {
                        writeFile file: 'reports/sonar-scan.log', text: "SonarQube Scan failed before completion.\\n\\n${err.message}\\n"
                        throw err
                    } finally {
                        sh '''#!/usr/bin/env bash
node -e '
const fs = require("fs");
const logPath = "reports/sonar-scan.log";
const taskPath = ".scannerwork/report-task.txt";
const htmlPath = "reports/sonar-scan.html";
const escapeHtml = (value) => value
  .replace(/&/g, "&amp;")
  .replace(/</g, "&lt;")
  .replace(/>/g, "&gt;")
  .replace(/"/g, "&quot;");
const log = fs.existsSync(logPath) ? fs.readFileSync(logPath, "utf8") : "No SonarScanner log was generated.";
const task = fs.existsSync(taskPath) ? fs.readFileSync(taskPath, "utf8") : "";
const dashboardUrl = (task.match(/^dashboardUrl=(.+)$/m) || [])[1] || "";
const html = `<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>SonarQube Scan Report</title>
    <style>
      body { font-family: Arial, sans-serif; margin: 24px; color: #17212b; }
      h1 { margin-bottom: 8px; }
      .meta { margin-bottom: 18px; color: #5d6b78; }
      pre { white-space: pre-wrap; border: 1px solid #d8e0e8; border-radius: 6px; padding: 14px; background: #f6f8fa; }
      a { color: #14796f; }
    </style>
  </head>
  <body>
    <h1>SonarQube Scan Report</h1>
    <div class="meta">Project: student-management-system</div>
    ${dashboardUrl ? `<p><a href="${escapeHtml(dashboardUrl)}">Open SonarQube Dashboard</a></p>` : ""}
    ${task ? `<h2>Scanner Task</h2><pre>${escapeHtml(task)}</pre>` : ""}
    <h2>Scanner Output</h2>
    <pre>${escapeHtml(log)}</pre>
  </body>
</html>`;
fs.writeFileSync(htmlPath, html);
'
'''
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/sonar-scan.html,reports/sonar-scan.log,.scannerwork/report-task.txt', allowEmptyArchive: true
                }
            }
        }

        stage('Deploy to SSH Server') {
            steps {
                sshagent(['webserver1']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no "$SERVER" 'bash -lc "
                            set -e
                            cd '"'"'$APP_DIR'"'"'
                            git pull origin '"'"'$BRANCH'"'"'
                            if ! command -v pm2 >/dev/null 2>&1; then
                                [ -s \\"$HOME/.nvm/nvm.sh\\" ] && . \\"$HOME/.nvm/nvm.sh\\"
                            fi
                            if ! command -v pm2 >/dev/null 2>&1; then
                                npm install -g pm2 || sudo npm install -g pm2
                            fi
                            pm2 restart 0
                        "'
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Tests passed and deployment completed successfully.'
        }

        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
