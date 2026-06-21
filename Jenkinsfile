pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

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
                withCredentials([
                    file(credentialsId: 'vault.pass', variable: 'VAULT_FILE'),
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                    ansible --version

                    ANSIBLE_STDOUT_CALLBACK=default \
                    ansible-inventory -i $INVENTORY --list \
                    --vault-password-file $VAULT_FILE
                    '''
                }
            }
        }

        stage('Syntax Check') {
            steps {
                withCredentials([
                    file(credentialsId: 'vault.pass', variable: 'VAULT_FILE'),
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                    ANSIBLE_STDOUT_CALLBACK=default \
                    ansible-playbook $PLAYBOOK \
                    -i $INVENTORY \
                    --syntax-check \
                    --vault-password-file $VAULT_FILE
                    '''
                }
            }
        }

        stage('Pre Check (Ping Test)') {
            steps {
                withCredentials([
                    file(credentialsId: 'vault.pass', variable: 'VAULT_FILE'),
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                    ANSIBLE_STDOUT_CALLBACK=default \
                    ansible all \
                    -i $INVENTORY \
                    -m ping \
                    -u $SSH_USER \
                    --private-key $SSH_KEY \
                    --vault-password-file $VAULT_FILE
                    '''
                }
            }
        }

        stage('Deploy Patch') {
            steps {
                withCredentials([
                    file(credentialsId: 'vault.pass', variable: 'VAULT_FILE'),
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                    ANSIBLE_STDOUT_CALLBACK=default \
                    ansible-playbook $PLAYBOOK \
                    -i $INVENTORY \
                    -u $SSH_USER \
                    --private-key $SSH_KEY \
                    --vault-password-file $VAULT_FILE \
                    -vv
                    '''
                }
            }
        }

        stage('Post Check') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                    ansible all \
                    -i $INVENTORY \
                    -m shell \
                    -a "uptime" \
                    -u $SSH_USER \
                    --private-key $SSH_KEY
                    '''
                }
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
