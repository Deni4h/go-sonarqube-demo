pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://192.168.1.10:8200' // Ganti sesuai alamat Vault kamu
        VAULT_SECRET = 'secret/sonarqube'
    }

    stages {
        stage('Fetch Secrets from Vault') {
            steps {
                script {
                    def vaultSecrets = vault path: "${VAULT_SECRET}"
                    env.SONAR_TOKEN = vaultSecrets.data.token
                    env.SONAR_HOST_URL = vaultSecrets.data.url
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git 'git@github.com:Deni4h/go-sonarqube-demo.git' // Ganti dengan repo kamu
            }
        }

        stage('SonarQube Scan') {
            environment {
                SONAR_SCANNER_HOME = tool 'SonarQubeScanner' // Ganti dengan nama scanner di Jenkins Tools
            }
            steps {
                withSonarQubeEnv('MySonar') {
                    sh """
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=go-demo-testing \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${env.SONAR_HOST_URL} \
                        -Dsonar.login=${env.SONAR_TOKEN}
                    """
                }
            }
        }

        // Optional deploy stage
        // stage('Deploy') {
        //     steps {
        //         echo "Deploying app..."
        //     }
        // }
    }
}
