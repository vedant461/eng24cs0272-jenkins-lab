pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 15, unit: 'MINUTES')
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        APP_NAME = 'eng24cs0272-calculator'
    }

    stages {
        stage('Build') {
            steps {
                sh 'python3 -m py_compile calculator.py'
                echo 'Python syntax check passed.'
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m pytest -v --junitxml=test-results.xml'
            }
        }

        stage('Package') {
            steps {
                sh 'mkdir -p dist'

                sh 'tar -czf dist/${APP_NAME}-${BUILD_NUMBER}.tar.gz calculator.py'

                archiveArtifacts artifacts: 'dist/*.tar.gz',
                    fingerprint: true
            }
        }

        stage('Docker image') {
            steps {
                sh 'docker build -t ${APP_NAME}:${BUILD_NUMBER} .'

                sh 'docker run --rm ${APP_NAME}:${BUILD_NUMBER}'

                sh 'docker images ${APP_NAME}'
            }
        }

        stage('Deploy to staging') {
            steps {
                input message: 'Deploy to staging?',
                      ok: 'Deploy'

                sh 'mkdir -p /tmp/eng24cs0272-staging'

                sh 'cp calculator.py /tmp/eng24cs0272-staging/'

                sh 'ls -l /tmp/eng24cs0272-staging'
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true,
                  testResults: 'test-results.xml'
        }

        success {
            echo "Build ${env.BUILD_NUMBER} of ${env.APP_NAME} passed."
        }

        failure {
            echo "Build ${env.BUILD_NUMBER} failed. Check Console Output."
        }
    }
}
