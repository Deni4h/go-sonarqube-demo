pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://192.168.1.11:8200'       // Alamat Vault
        VAULT_SECRET = 'secret/sonarqube'        // Path secret Vault (jangan lupa /data/ jika pakai KV v2)
        VAULT_TOKEN = 'hvs.vN8MOWnQdDUVD5r4fVO5Rjlt'   // Token Vault
    }

    stages {
        stage('Fetch Secrets from Vault') {
            steps {
                sh '''
                    echo "Mengambil secret dari Vault..."

                    # Request ke Vault
                    RESPONSE=$(curl -s $VAULT_ADDR/v1/$VAULT_SECRET -H "X-Vault-Token: $VAULT_TOKEN")

                    # Ambil nilai dari key `token` dan `url`
                    SONAR_TOKEN=$(echo "$RESPONSE" | jq -r '.data.data.token')
                    SONAR_HOST_URL=$(echo "$RESPONSE" | jq -r '.data.data.url')

                    # Simpan ke file agar bisa digunakan di stage lain
                    echo "SONAR_TOKEN=$SONAR_TOKEN" > vault.env
                    echo "SONAR_HOST_URL=$SONAR_HOST_URL" >> vault.env

                    echo "Secrets berhasil diambil dan disimpan."
                '''
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

                    // Load secrets dari file vault.env
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
