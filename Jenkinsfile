pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
        timeout(time: 5, unit: 'MINUTES')
    }

    environment {
        FORCE_COLOR = '0'
        NO_COLOR = 'true'
    }

    stages {
        stage('Audit tools') {
            steps {
                sh 'node --version'
                sh 'npm --version'
            }
        }

        stage('Install dependencies') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }

        stage('Generate files') {
            steps {
                dir('backend') {
                    sh 'npm run prisma:generate'
                }
            }
        }

        stage('Format check') {
            steps {
                dir('backend') {
                    sh 'npm run format:check'
                }
            }
        }

        stage('Code quality') {
            steps {
                dir('backend') {
                    sh 'npm run lint'
                }
            }
        }

        stage('Type check') {
            steps {
                dir('backend') {
                    sh 'npm run type-check'
                }
            }
        }

        stage('Tests') {
            steps {
                dir('backend') {
                    sh 'npm run test'
                }
            }
        }

        stage('Build') {
            steps {
                dir('backend') {
                    sh 'npm run build'
                    // Al estar dentro de dir('backend'), archivará correctamente 'backend/dist/**'
                    archiveArtifacts artifacts: 'dist/**', fingerprint: true
                }
            }
        }
    }

    post {
        always {
            // Garantiza que el espacio de trabajo se limpie termine como termine el job
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Review logs.'
        }
    }
}