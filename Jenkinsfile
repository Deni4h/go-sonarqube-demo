pipeline {
    agent any

    environment {
        VAULT_ADDR = 'http://192.168.1.11:8200'
        VAULT_SECRET = 'secret/sonarqube' // KV v1 (bukan KV v2, jadi tidak perlu /data/)
    }

    stages {
        stage('Fetch Secrets from Vault') {
             environment {
                VAULT_TOKEN = credentials('vault-root-token') // Ambil dari Jenkins Credentials
            }
            steps {
                sh '''
                    echo "Mengambil secret dari Vault KV v1..."

                    RESPONSE=$(curl -s $VAULT_ADDR/v1/$VAULT_SECRET -H "X-Vault-Token: $VAULT_TOKEN")

                    SONAR_TOKEN=$(echo "$RESPONSE" | jq -r '.data.token')
                    SONAR_HOST_URL=$(echo "$RESPONSE" | jq -r '.data.url')

                    echo "SONAR_TOKEN=${SONAR_TOKEN}" > vault.env
                    echo "SONAR_HOST_URL=${SONAR_HOST_URL}" >> vault.env

                    echo "Secrets berhasil diambil:"
                    cat vault.env
                '''
            }
        }

        stage('Checkout Code') {
            steps {
                git(
                    url: 'git@github.com:Deni4h/go-sonarqube-demo.git',
                    branch: 'main',
                    credentialsId: 'git'
                )
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    // Baca secret dari vault.env
                    def props = readProperties file: 'vault.env'

                    def SONAR_TOKEN = props['SONAR_TOKEN']
                    def SONAR_HOST_URL = props['SONAR_HOST_URL']

                    // Ambil path sonar scanner yang sudah didefinisikan di Jenkins
                    def scannerHome = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                    // Eksekusi scan
                    sh """
                        ${scannerHome}/bin/sonar-scanner \\
                        -Dsonar.projectKey=go-demo-testing \\
                        -Dsonar.sources=. \\
                        -Dsonar.host.url=${SONAR_HOST_URL} \\
                        -Dsonar.login=${SONAR_TOKEN}
                    """
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
