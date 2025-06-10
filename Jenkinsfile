pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://192.168.1.10:8200' // Alamat Vault
        VAULT_SECRET = 'secret/sonarqube'
    }

    stages {
        stage('Fetch Secrets from Vault') {
            steps {
                script {
                    def secrets = vault(
                        vaultUrl: env.VAULT_ADDR,
                        vaultCredentialId: 'vault-jenkins-token', // ID kredensial yang sudah dibuat di Jenkins
                        engineVersion: 2,
                        path: "${VAULT_SECRET}",
                        secretValues: [
                            [envVar: 'SONAR_TOKEN', vaultKey: 'token'],
                            [envVar: 'SONAR_HOST_URL', vaultKey: 'url']
                        ]
                    )
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git 'git@github.com:Deni4h/go-sonarqube-demo.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv('MySonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \\
                            -Dsonar.projectKey=go-demo-testing \\
                            -Dsonar.sources=. \\
                            -Dsonar.host.url=$SONAR_HOST_URL \\
                            -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        // stage('Deploy') {
        //     steps {
        //         echo "Deploying app..."
        //     }
        // }
    }
}
