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
                    waitForQualityGate abortPipeline: true
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
                sh './gradlew publish -x test'
            }
        }
    }
}



//     post {
//         failure {
//             echo ' Pipeline failed! Sending failure notifications...'
//             emailext (
//                 subject: "Build Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
//                 body: """
//                     <h2 style="color: red;"> Build Failed!</h2>
//                     <p><strong>Job:</strong> ${env.JOB_NAME}</p>
//                     <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
//                     <p><strong>Failed Stage:</strong> Check console output</p>
//                     <p><a href="${env.BUILD_URL}console">View Console Output</a></p>
//                     <p><a href="${env.BUILD_URL}">View Build</a></p>
//                 """,
//                 to: "${env.RECIPIENT_EMAIL}",
//                 mimeType: 'text/html'
//             )
//         }
//         success {
//             echo ' Pipeline completed successfully!'
//             emailext (
//                 subject: " Build Success - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
//                 body: """
//                     <h2 style="color: green;"> Build Successful!</h2>
//                     <p><strong>Job:</strong> ${env.JOB_NAME}</p>
//                     <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
//                     <h3> Available Reports:</h3>
//                     <ul>
//                         <li><a href="${env.BUILD_URL}Unit_20Test_20Report/">Unit Test Report</a></li>
//                         <li><a href="${env.BUILD_URL}Jacoco_20Coverage_20Report/">Jacoco Coverage Report</a></li>
//                         <li><a href="${env.BUILD_URL}Javadoc/">Javadoc</a></li>
//                     </ul>
//                     <p><a href="${env.BUILD_URL}">View Full Build</a></p>
//                 """,
//                 to: "${env.RECIPIENT_EMAIL}",
//                 mimeType: 'text/html'
//             )
//         }
//     }

