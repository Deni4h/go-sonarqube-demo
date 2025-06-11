pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://192.168.1.11:8200'
        VAULT_SECRET = 'secret/sonarqube'   // KV v2: tambahkan /data/
        VAULT_TOKEN = 'hvs.vN8MOWnQdDUVD5r4fVO5Rjlt'
    }

    stages {
        stage('Fetch Secrets from Vault') {
            steps {
                sh '''
                    echo "Mengambil secret dari Vault..."

                    RESPONSE=$(curl -s $VAULT_ADDR/v1/$VAULT_SECRET -H "X-Vault-Token: $VAULT_TOKEN")

                    SONAR_TOKEN=$(echo "$RESPONSE" | jq -r '.data.token')
                    SONAR_HOST_URL=$(echo "$RESPONSE" | jq -r '.data.url')

                    echo "SONAR_TOKEN=$SONAR_TOKEN" > vault.env
                    echo "SONAR_HOST_URL=$SONAR_HOST_URL" >> vault.env

                    echo "Secrets berhasil diambil dan disimpan."
                '''
            }
        }

        stage('Checkout Code') {
            steps {
                git(
                    url: 'git@github.com:Deni4h/go-sonarqube-demo.git',
                    branch: 'main',
                    credentialsId: 'git' // Ganti dengan ID credentials SSH kamu
                )
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                    def props = readProperties file: 'vault.env'
                    env.SONAR_TOKEN = props['SONAR_TOKEN']
                    env.SONAR_HOST_URL = props['SONAR_HOST_URL']

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
