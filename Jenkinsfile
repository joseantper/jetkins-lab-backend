pipeline {
    agent any
    
    stages {
        stage('Audit tools') {
            steps {
                // Aquí no pasa nada si se ejecuta en la raíz
                sh 'node --version'
                sh 'npm --version'
            }
        }

        stage('Install dependencies') {
            steps {
                // 📂 Le decimos a Jenkins que entre a la carpeta backend
                dir('backend') {
                    sh 'npm install'
                }
            }
        }
        
        stage('Format check') {
            steps {
                dir('backend') {
                    // Aquí el comando que uses, por ejemplo: sh 'npm run lint'
                }
            }
        }

        // Repite el bloque dir('backend') { ... } en los demás stages 
        // (Tests, Build, etc.) que necesiten interactuar con tu código Node.
    }
}