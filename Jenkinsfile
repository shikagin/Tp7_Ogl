pipeline {
    agent any

    stages {
        stage('Test & Code Analysis') {
            steps {
                echo 'Running tests and SonarQube analysis...'

                // Run tests and generate reports first
                sh './gradlew clean test jacocoTestReport'

                // Wrap SonarQube analysis with withSonarQubeEnv
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh './gradlew sonar'  // Changed from mvn to gradlew
                    }
                }

                junit '**/build/test-results/test/*.xml'

                // Publish Unit Test Report
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'build/reports/tests/test',
                    reportFiles: 'index.html',
                    reportName: 'Unit Test Report'
                ])

                // Publish Jacoco Coverage Report
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'build/reports/jacoco/html',
                    reportFiles: 'index.html',
                    reportName: 'Jacoco Coverage Report'
                ])
            }
        }

        stage('Code Quality') {
            steps {
                echo '========== Phase Code Quality =========='
                // Vérification du Quality Gate - arrêt en cas de Failed
                timeout(time: 5, unit: 'MINUTES') {
//                     waitForQualityGate abortPipeline: true
                    echo 'SonarQube analysis submitted! View at: http://localhost:9000/dashboard?id=tp5'
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building JAR and documentation...'
                sh './gradlew build -x test'
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
                echo 'Deploying to Maven repository...'
                sh './gradlew publish -x test --no-daemon'
            }
        }
    }
}
post {
    success {
        script {
            // Email notification
            catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                emailext(
                    to: 'mr_mekircha@esi.dz',
                    subject: "✅ Jenkins SUCCESS - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        Build succeeded!

                        Job: ${env.JOB_NAME}
                        Build: #${env.BUILD_NUMBER}
                        URL: ${env.BUILD_URL}

                        Duration: ${currentBuild.durationString}
                    """
                )
            }

            // Slack notification
            catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                slackSend(
                    channel: '#jenkins',
                    color: 'good',
                    message: "✅ Build SUCCESS\nJob: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}"
                )
            }
        }
    }

    failure {
        script {
            // Email notification
            catchError(buildResult: 'FAILURE', stageResult: 'UNSTABLE') {
                emailext(
                    to: 'mr_mekircha@esi.dz',
                    subject: "❌ Jenkins FAILURE - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        Build failed!

                        Job: ${env.JOB_NAME}
                        Build: #${env.BUILD_NUMBER}
                        URL: ${env.BUILD_URL}

                        Please check the console output.
                    """
                )
            }

            // Slack notification
            catchError(buildResult: 'FAILURE', stageResult: 'UNSTABLE') {
                slackSend(
                    channel: '#jenkins',
                    color: 'danger',
                    message: "❌ Build FAILURE\nJob: ${env.JOB_NAME} #${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}"
                )
            }
        }
    }
}


