pipeline {
    agent any

    parameters {
        booleanParam(name: 'SKIP_QUALITY', defaultValue: false, description: 'Skip Code Quality Analysis')
        booleanParam(name: 'SKIP_COVERAGE', defaultValue: false, description: 'Skip Code Coverage Analysis')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip Stability Tests')
    }

    environment {
        MAVEN_HOME = '/usr/share/maven'  // adjust based on your system
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/Abhishek-bisht-tech/java-ci-demo.git'
            }
        }

        stage('Parallel Checks') {
            parallel {
                stage('Code Stability (Tests)') {
                    when { not { expression { params.SKIP_TESTS } } }
                    steps {
                        sh 'mvn clean test'
                    }
                }

                stage('Code Quality Analysis') {
                    when { not { expression { params.SKIP_QUALITY } } }
                    steps {
                        withSonarQubeEnv('MySonarQube') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }

                stage('Code Coverage Analysis') {
                    when { not { expression { params.SKIP_COVERAGE } } }
                    steps {
                        sh 'mvn clean test jacoco:report'
                    }
                }
            }
        }

        stage('Publish Reports') {
            steps {
                publishHTML(target: [
                    reportDir: 'target/site/jacoco',
                    reportFiles: 'index.html',
                    reportName: 'JaCoCo Coverage Report'
                ])
            }
        }

        stage('Approval to Publish') {
            steps {
                script {
                    def userInput = input(
                        message: "Do you want to proceed with artifact publishing?",
                        parameters: [choice(choices: ['Yes', 'No'], description: 'Choose', name: 'Confirm')]
                    )
                    if (userInput == 'No') {
                        error "Artifact publishing was denied"
                    }
                }
            }
        }

        stage('Publish Artifacts') {
            steps {
                sh 'mvn clean install'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            mail to: 'abhishekbisht321@gmail.com',
                 subject: '✅ Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                 body: """\
The Jenkins build was successful.

Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
View here: ${env.BUILD_URL}
"""

            slackSend(channel: '#build-status', message: "✅ Build #${env.BUILD_NUMBER} succeeded for ${env.JOB_NAME}")
        }
        failure {
            mail to: 'abhishekbisht321@gmail.com',
                 subject: '❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                 body: """\
The Jenkins build failed.

Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
View logs: ${env.BUILD_URL}
"""

            slackSend(channel: '#build-status', message: "❌ Build #${env.BUILD_NUMBER} failed for ${env.JOB_NAME}")
        }
    }
}
