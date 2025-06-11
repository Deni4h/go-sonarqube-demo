pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://192.168.1.11:8200' // Alamat Vault
        VAULT_SECRET = 'secret/sonarqube'
        VAULT_TOKEN = 'hvs.vN8MOWnQdDUVD5r4fVO5Rjlt' // Token Vault
    }

    stages {
        stage('Fetch Secrets via curl') {
            steps {
                sh '''
                    echo "Mengambil secret langsung dari Vault via API:"
                    curl $VAULT_ADDR/v1/$VAULT_SECRET \
                      -H "X-Vault-Token: $VAULT_TOKEN"
                '''
            }
        }

        stage('Fetch Secrets from Vault Plugin') {
            steps {
                script {
                    withVault([
                        vaultSecrets: [[
                            path: "${VAULT_SECRET}",
                            secretValues: [
                                [envVar: 'SONAR_TOKEN', vaultKey: 'token'],
                                [envVar: 'SONAR_HOST_URL', vaultKey: 'url']
                            ]
                        ]],
                        vaultUrl: env.VAULT_ADDR,
                        credentialsId: 'vault-jenkins-token',
                        engineVersion: 2
                    ]) {
                        sh '''
                            echo "Fetched SONAR_TOKEN: $SONAR_TOKEN"
                            echo "Fetched SONAR_HOST_URL: $SONAR_HOST_URL"
                        '''
                    }
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
