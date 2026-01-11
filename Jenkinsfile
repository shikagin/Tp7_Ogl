pipeline {
    agent any

    stages {
        stage('Test & Code Analysis') {
            steps {
                echo 'Running tests and SonarQube analysis...'

                // Run tests and generate reports first
                sh './gradlew clean test jacocoTestReport'

                // Wrap SonarQube analysis with withSonarQubeEnv
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean package sonar:sonar'
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
