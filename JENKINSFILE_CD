pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git url: 'https://github.com/albertmenendezg/UNIR-CP1.D', branch: 'master'
                sh '''
                    git clone --branch production https://github.com/albertmenendezg/UNIR-CP1.D-config.git config
                    cp config/samconfig.toml .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region us-east-1
                    sam deploy --no-confirm-changeset --force-upload --no-fail-on-empty-changeset --no-progressbar
                '''
            }
        }

        stage('Rest Test') {
            
            environment {
                BASE_URL = 'https://6rwz7kh6j2.execute-api.us-east-1.amazonaws.com/Prod'
            }

            steps {
                sh '''
                    pytest test/integration/todoApiTest.py -m read_only_api -v
                '''
            }
        }
    }
}
