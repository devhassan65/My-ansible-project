pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Ansible Collections') {
            steps {
                sh '''
                    ansible-galaxy collection install -r requirements.yml
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY'
                    ),
                    string(
                        credentialsId: 'ansible-vault-password',
                        variable: 'VAULT_PASSWORD'
                    )
                ]) {
                    sh '''
                        chmod 400 "$SSH_KEY"

                        printf '%s\\n' "$VAULT_PASSWORD" > vault_pass.txt
                        chmod 600 vault_pass.txt

                        ansible-playbook \
                            -i inventory/hosts \
                            playbook/site.yml \
                            --private-key "$SSH_KEY" \
                            --vault-password-file vault_pass.txt

                        rm -f vault_pass.txt
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f vault_pass.txt || true'
        }
    }
}


