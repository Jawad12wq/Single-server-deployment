pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }

    stages {
        stage('Setup') {
            steps {
                sh '''
                python3 -m venv venv   # Create a virtual environment
                source venv/bin/activate  # Activate the virtual environment
                pip install --upgrade pip  # Upgrade pip to avoid potential issues
                pip install -r requirements.txt  # Install dependencies inside venv
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                source venv/bin/activate
                pytest
                '''
            }
        }

        stage('Package code') {
            steps {
                sh '''
                zip -r myapp.zip ./* -x '*.git*'
                ls -lart
                '''
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip ${username}@${SERVER_IP}:/home/ec2-user/
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app/
                        python3 -m venv /home/ec2-user/app/venv
                        source /home/ec2-user/app/venv/bin/activate
                        cd /home/ec2-user/app/
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        sudo systemctl restart flaskapp.service
EOF
                    '''
                }
            }
        }
    }
}
