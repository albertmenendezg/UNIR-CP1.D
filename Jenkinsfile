pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git url: 'https://github.com/albertmenendezg/UNIR-CP1.D', branch: 'develop'
                sh '''
                    git clone --branch staging https://github.com/albertmenendezg/UNIR-CP1.D-config.git config
                    cp config/samconfig.toml .
                '''
            }
        }

        stage('Static Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh '''
                        mkdir -p reports
                        bandit --exit-zero -r src -f custom -o reports/bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                        flake8 --exit-zero --format=pylint src > reports/flake8.out
                    '''

                    recordIssues tools: [
                        pyLint(name: 'Bandit', pattern: 'reports/bandit.out'),
                        flake8(name: 'Flake8', pattern: 'reports/flake8.out')
                    ]
                }
            }

            post {
                always {
                    archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
                }
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
