pipeline {
    agent any

    parameters {
        booleanParam(name: 'SKIP_STABILITY_CHECK', defaultValue: false, description: 'Skip Code Stability Check')
        booleanParam(name: 'SKIP_QUALITY_CHECK', defaultValue: false, description: 'Skip Code Quality Check')
        booleanParam(name: 'SKIP_COVERAGE_CHECK', defaultValue: false, description: 'Skip Code Coverage Check')
    }

    environment {
        EMAIL_RECIPIENT = 'abhishekbisht321@gmail.com'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Parallel Scans') {
            parallel {
                stage('Code Stability') {
                    when {
                        expression { return !params.SKIP_STABILITY_CHECK }
                    }
                    steps {
                        echo 'Running unit tests for stability...'
                        sh './gradlew test' // or mvn test
                        junit '**/build/test-results/test/*.xml'
                    }
                }

                stage('Code Quality Analysis') {
                    when {
                        expression { return !params.SKIP_QUALITY_CHECK }
                    }
                    steps {
                        echo 'Running code quality analysis...'
                        sh './gradlew sonarqube' // assumes SonarQube config
                    }
                }

                stage('Code Coverage Analysis') {
                    when {
                        expression { return !params.SKIP_COVERAGE_CHECK }
                    }
                    steps {
                        echo 'Running code coverage analysis...'
                        sh './gradlew jacocoTestReport'
                        publishHTML(target: [
                            reportName : 'Code Coverage',
                            reportDir  : 'build/reports/jacoco/test/html',
                            reportFiles: 'index.html'
                        ])
                    }
                }
            }
        }

        stage('Approval to Publish') {
            steps {
                script {
                    def userInput = input(
                        message: 'Approve to publish artifacts?',
                        ok: 'Yes, publish',
                        parameters: [
                            choice(name: 'Approve', choices: ['Yes', 'No'], description: 'Approval to publish?')
                        ]
                    )

                    if (userInput == 'No') {
                        currentBuild.result = 'ABORTED'
                        error('Publication Denied')
                    }
                }
            }
        }

        stage('Publish Artifacts') {
            steps {
                echo 'Publishing artifacts...'
                sh './gradlew assemble' // or mvn package
                archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
            slackSend channel: '#ci', message: "✅ Build ${env.JOB_NAME} #${env.BUILD_NUMBER} Succeeded!"
            mail to: "${env.EMAIL_RECIPIENT}",
                 subject: "Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The build completed successfully. See details at ${env.BUILD_URL}"
        }

        failure {
            echo 'Pipeline failed.'
            slackSend channel: '#ci', message: "❌ Build ${env.JOB_NAME} #${env.BUILD_NUMBER} Failed!"
            mail to: "${env.EMAIL_RECIPIENT}",
                 subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The build failed. See details at ${env.BUILD_URL}"
        }

        aborted {
            echo 'Pipeline aborted by user.'
            slackSend channel: '#ci', message: "⚠️ Build ${env.JOB_NAME} #${env.BUILD_NUMBER} Aborted!"
        }
    }
}
