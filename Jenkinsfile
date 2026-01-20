pipeline {
    agent any

    stages {
        stage('Test & Code Analysis') {
            steps {
                echo 'Running tests and SonarQube analysis...'

                sh './gradlew clean test jacocoTestReport'

                script {
                    withSonarQubeEnv('SonarQube') {
                        sh './gradlew sonar'
                    }
                }

                junit '**/build/test-results/test/*.xml'

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
                    reportDir: 'build/reports/jacoco/html',
                    reportFiles: 'index.html',
                    reportName: 'Jacoco Coverage Report'
                ])
            }
        }

        stage('Code Quality') {
            steps {
                echo '========== Phase Code Quality =========='
                echo 'SonarQube analysis submitted! View at: http://localhost:9000/dashboard?id=tp5'
            }
        }

        stage('Build') {
            steps {
                echo 'Building JAR and documentation...'
                sh './gradlew build -x test'
                sh './gradlew javadoc'
                sh './gradlew sourcesJar javadocJar'

                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true

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

                // Slack notification with curl
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK')]) {
                        sh """
                            curl -X POST -H 'Content-type: application/json' \
                            --data '{
                                "text": "✅ *Build SUCCESS*",
                                "blocks": [
                                    {
                                        "type": "section",
                                        "text": {
                                            "type": "mrkdwn",
                                            "text": "*Status:* SUCCESS ✅\\n*Job:* ${env.JOB_NAME}\\n*Build:* #${env.BUILD_NUMBER}\\n*Duration:* ${currentBuild.durationString}\\n*URL:* ${env.BUILD_URL}"
                                        }
                                    }
                                ]
                            }' \
                            ${SLACK_WEBHOOK}
                        """
                    }
                }
            }
        }

        failure {

            script {
                // Email notification
                catchError(buildResult: 'FAILURE', stageResult: 'UNSTABLE') {
                    emailext(
                        to: 'mr_mekircha@esi.dz',
                        subject: " Jenkins FAILURE - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            Build failed!

                            Job: ${env.JOB_NAME}
                            Build: #${env.BUILD_NUMBER}
                            URL: ${env.BUILD_URL}

                            Please check the console output.
                        """
                    )
                }


                // Slack notification with curl
                catchError(buildResult: 'FAILURE', stageResult: 'UNSTABLE') {
                    withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK')]) {
                        sh """
                            curl -X POST -H 'Content-type: application/json' \
                            --data '{
                                "text": "❌ *Build FAILURE*",
                                "blocks": [
                                    {
                                        "type": "section",
                                        "text": {
                                            "type": "mrkdwn",
                                            "text": "*Status:* FAILURE ❌\\n*Job:* ${env.JOB_NAME}\\n*Build:* #${env.BUILD_NUMBER}\\n*URL:* ${env.BUILD_URL}\\n\\nPlease check the console output."
                                        }
                                    }
                                ]
                            }' \
                            ${SLACK_WEBHOOK}
                        """
                    }
                }
            }
        }
    }
}
