def secrets = [
    [
        path: 'secrets/creds/my-secret-text',
        engineVersion: 2,
        secretValues: [
            [envVar: 'SONARQUBE_TOKEN', vaultKey: 'currency']
        ]
    ]
]

def configuration = [
    vaultUrl: 'http://65.0.30.51:8200',
    vaultCredentialId: 'vault-geetha-token',
    engineVersion: 2
]
pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    
    DOCKERHUB_CREDENTIALS = credentials('dockerhub')
  }
  stages {
        stage('Vault') {
            steps {
                script {
                    withVault([configuration: configuration, vaultSecrets: currency]) {
                        // Extract the SonarQube Token
//                        SONARQUBE_TOKEN = env.SONARQUBE_TOKEN
                        sh "echo ${SONARQUBE_TOKEN}"
                    }
                }
            }

        }
  }
          stage('Docker Bench Security') {
      steps {
        sh 'chmod +x docker-bench-security.sh'
        sh './docker-bench-security.sh'
      }
    }


     stage('SonarQube Analysis') {
          agent any
      steps {
            withVault([configuration: configuration, vaultSecrets: secrets]) {
       sh '/var/opt/sonar-scanner-4.7.0.2747-linux/bin/sonar-scanner -Dsonar.projectKey=currencyservice-- -Dsonar.sources=. -Dsonar.host.url=http://172.31.7.193:9000 -Dsonar.token=$SONARQUBE_TOKEN'

        
      }
    }





    stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build('koushiksai/currencyservice:latest', '.')
                }
            }
        }


    stage('Login') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push koushiksai/currencyservice'
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}
