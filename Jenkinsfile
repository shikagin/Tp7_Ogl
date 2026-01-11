pipeline {
    agent any

     stages {
            stage('Test & Code Analysis') {
                steps {
                    sh './gradlew clean test'

}
}
}
}
//     environment {
//         GMAIL_USERNAME = credentials('gmail-username')
//         GMAIL_APP_PASSWORD = credentials('gmail-app-password')
//         RECIPIENT_EMAIL = credentials('recipient-email')
//         SLACK_WEBHOOK_URL = credentials('slack-webhook-url')
//         SONAR_TOKEN = credentials('sonarqube-token')
//     }
//
//     stages {
//         stage('Test & Code Analysis') {
//             steps {
//                 echo '🧪 Running tests and SonarQube analysis...'
//
//                 sh './gradlew clean test jacocoTestReport sonar'
//
//
//                 junit '**/build/test-results/test/*.xml'
//
//
//                 publishHTML([
//                     allowMissing: true,
//                     alwaysLinkToLastBuild: true,
//                     keepAll: true,
//                     reportDir: 'build/reports/tests/test',
//                     reportFiles: 'index.html',
//                     reportName: 'Unit Test Report'
//                 ])
//
//                 publishHTML([
//                     allowMissing: true,
//                     alwaysLinkToLastBuild: true,
//                     keepAll: true,
//                     reportDir: 'build/reports/jacoco',
//                     reportFiles: 'index.html',
//                     reportName: 'Jacoco Coverage Report'
//                 ])
//             }
//         }
//
//         stage('Quality Gate') {
//             steps {
//                 echo '✅ Checking Quality Gate...'
//                 timeout(time: 5, unit: 'MINUTES') {
//                     waitForQualityGate abortPipeline: true
//                 }
//             }
//         }
//
//         stage('Build') {
//             steps {
//                 echo '🏗️ Building JAR and documentation...'
//                 sh './gradlew build -x test'  // Skip tests since we already ran them
//                 sh './gradlew javadoc'
//                 sh './gradlew sourcesJar javadocJar'
//
//                 archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
//
//                 publishHTML([
//                     allowMissing: true,
//                     alwaysLinkToLastBuild: true,
//                     keepAll: true,
//                     reportDir: 'build/docs/javadoc',
//                     reportFiles: 'index.html',
//                     reportName: 'Javadoc'
//                 ])
//             }
//         }
//
//         stage('Deploy') {
//             steps {
//                 echo '🚀 Deploying to Maven repository...'
//                 sh './gradlew publish -x test'  // Skip tests
//             }
//         }
//
// //         stage('Notification') {
// //             steps {
// //                 echo '📧 Sending notifications...'
// //                 sh './gradlew notifyEmail'
// //                 sh './gradlew notifySlack'
// //             }
// //         }
//     }
//
// //     post {
// //         failure {
// //             echo '❌ Pipeline failed! Sending failure notifications...'
// //             emailext (
// //                 subject: "Build Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
// //                 body: """
// //                     <h2 style="color: red;">❌ Build Failed!</h2>
// //                     <p><strong>Job:</strong> ${env.JOB_NAME}</p>
// //                     <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
// //                     <p><strong>Failed Stage:</strong> Check console output</p>
// //                     <p><a href="${env.BUILD_URL}">View Build</a></p>
// //                 """,
// //                 to: "${env.RECIPIENT_EMAIL}",
// //                 mimeType: 'text/html'
// //             )
// //         }
// //         success {
// //             echo '✅ Pipeline completed successfully!'
// //         }
// //     }
// }