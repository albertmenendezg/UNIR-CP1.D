pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git url: 'https://github.com/albertmenendezg/UNIR-CP1.D', branch: 'develop'
                sh '''
                    rm -rf config
                    git clone --branch staging https://github.com/albertmenendezg/UNIR-CP1.D-config.git config
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
                BASE_URL = 'https://biyt4fra52.execute-api.us-east-1.amazonaws.com/Prod'
            }
            steps {
                sh '''
                    pytest test/integration/todoApiTest.py
                '''
            }
        }

        stage('Promote') {
            steps {
                sshagent(credentials: ['github-ssh-key']) {
                    sh '''
                        git remote set-url origin git@github.com:albertmenendezg/UNIR-CP1.D.git
                        git checkout master
                        git pull
                        git merge develop
                        git push origin master
                    '''
                }
            }
        }
    }
}
