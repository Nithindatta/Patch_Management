pipeline {
    agent any

    environment {
        INVENTORY = "/etc/ansible/inventory.ini"
        PLAYBOOK  = "patching.yaml"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'git',
                    url: 'git@github.com:Nithindatta/Patch_Management.git'
            }
        }

        stage('Validate Ansible Setup') {
            steps {
                sh '''
                ansible --version
                ansible-inventory -i $INVENTORY --list
                '''
            }
        }

        stage('Syntax Check') {
            steps {
                withCredentials([file(credentialsId: 'vault.pass', variable: 'VAULT_FILE')]) {
                    sh '''
                    ANSIBLE_STDOUT_CALLBACK=default ansible-playbook $PLAYBOOK -i $INVENTORY \
                    --syntax-check \
                    --vault-password-file $VAULT_FILE
                    '''
                }
            }
        }

        stage('Pre Check (Ping Test)') {
            steps {
                sh '''
                ansible all -i $INVENTORY -m ping
                '''
            }
        }

        stage('Deploy Patch') {
            steps {
                withCredentials([file(credentialsId: 'vault.pass', variable: 'VAULT_FILE')]) {
                    sh '''
                    ANSIBLE_STDOUT_CALLBACK=default ansible-playbook $PLAYBOOK -i $INVENTORY \
                    --vault-password-file $VAULT_FILE -vv
                    '''
                }
            }
        }

        stage('Post Check') {
            steps {
                sh '''
                ansible all -i $INVENTORY -m shell -a "uptime"
                '''
            }
        }
    }

    post {
        success {
            echo "Patching Completed Successfully"
        }

        failure {
            echo "Patching Failed - Check Logs"
        }
    }
}
