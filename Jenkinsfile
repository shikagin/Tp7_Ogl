pipeline {
    agent any

    environment {
        GMAIL_USERNAME = credentials('gmail-username')
        GMAIL_APP_PASSWORD = credentials('Gmail App Password')
        RECIPIENT_EMAIL = credentials('recipient-email')
        SLACK_WEBHOOK_URL = credentials('slack-webhook-url')
    }

    stages {
        stage('Test') {
            steps {
                echo '🧪 Running unit tests...'
                sh './gradlew test'
                sh './gradlew jacocoTestReport'
                sh './gradlew cucumberReports'

                // Archive test results
                junit '**/build/test-results/test/*.xml'

                // Publish HTML reports with required parameters
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'build/reports/tests/test',
                    reportFiles: 'index.html',
                    reportName: 'Unit Test Report'
                ])
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'build/reports/cucumber',
                    reportFiles: 'cucumber-html-reports/overview-features.html',
                    reportName: 'Cucumber Report'
                ])
            }
        }

        stage('Code Analysis') {
            steps {
                echo '🔍 Running SonarQube analysis...'
                sh './gradlew sonar'
            }
        }

        stage('Quality Gate') {
            steps {
                echo '✅ Checking Quality Gate...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                echo '🏗️ Building JAR and documentation...'
                sh './gradlew clean build'
                sh './gradlew javadoc'
                sh './gradlew sourcesJar javadocJar'

                // Archive artifacts
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true

                // Publish Javadoc
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'build/docs/javadoc',
                    reportFiles: 'index.html',
                    reportName: 'Javadoc'
                ])
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Deploying to Maven repository...'
                sh './gradlew publish'
            }
        }

        stage('Notification') {
            steps {
                echo '📧 Sending notifications...'
                sh './gradlew notifyEmail'
                sh './gradlew notifySlack'
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed! Sending failure notifications...'
            emailext (
                subject: "Build Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2 style="color: red;">❌ Build Failed!</h2>
                    <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
                    <p><strong>Failed Stage:</strong> Check console output</p>
                    <p><a href="${env.BUILD_URL}">View Build</a></p>
                """,
                to: "${env.RECIPIENT_EMAIL}",
                mimeType: 'text/html'
            )
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
    }
}
