pipeline {
    agent any

    stages {
        stage("Install Ansible") {
            steps {
                sh 'python3 -m pip install ansible requests docker'
            }
        }
        stage("Generate Ansible Inventory") {
            steps {
                sh 'aws ec2 describe-instances --region eu-north-1 --filters "Name=tag:App,Values=AlonitBot"  --query "Reservations[].Instances[]" > hosts.json'
                sh 'python3 prepare_ansible_inv.py'
                sh '''
                echo "Inventory generated"
                cat hosts
                '''
            }
        }
        stage('Ansible Bot Deploy') {
            environment {
                ANSIBLE_HOST_KEY_CHECKING = 'False'
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'bot-machine', usernameVariable: 'ssh_user', keyFileVariable: 'privatekey')]) {
                    sh '''
                    /var/lib/jenkins/.local/bin/ansible-playbook botDeploy.yaml -e 'ansible_python_interpreter=/usr/bin/python3' --extra-vars "bot_image=$BOT_IMAGE" --user=${ssh_user} -i hosts --private-key ${privatekey}
                    '''
                }
            }
        }
    }
}