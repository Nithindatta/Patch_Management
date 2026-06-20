pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Syntax Check') {
            steps {
                withCredentials([file(credentialsId: 'vault.pass', variable: 'VAULT_FILE')]) {
                    sh '''
                    ansible-playbook patching.yaml \
                    -i inventory.ini \
                    --syntax-check \
                    --vault-password-file $VAULT_FILE
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: 'vault-pass', variable: 'VAULT_FILE')]) {
                    sh '''
                    ansible-playbook patching.yaml \
                    -i inventory.ini \
                    --vault-password-file $VAULT_FILE
                    '''
                }
            }
        }
    }
}
