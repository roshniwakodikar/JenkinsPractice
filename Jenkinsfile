pipeline {
    agent any
    environment {
        AWS_CREDENTIALS = credentials('AWS Admin User')  // AWS credentials stored in Jenkins
        GIT_CREDENTIALS = credentials('GitHub Personal Token')  // GitHub token stored in Jenkins
    }
    stages {
        stage('Checkout SCM') {
            steps {
                // Checkout the CloudFormation template and Ansible playbook from GitHub
                checkout([$class: 'GitSCM', branches: [[name: '*/dev']],
                userRemoteConfigs: [[url: 'https://github.com/roshniwakodikar/JenkinsPractice.git', credentialsId: 'GitHub Personal Token']]])
            }
        }
        stage('Deploy CloudFormation Stack') {
            steps {
                script {
                    // Use AWS CLI to create the CloudFormation stack for EC2
                    bat '''
                        powershell -Command "aws cloudformation create-stack --stack-name MyEC2InstanceStack --template-body file://cloudformation_ec2.yaml --parameters ParameterKey=InstanceType,ParameterValue=t2.micro --capabilities CAPABILITY_IAM"
                    '''
                }
            }
        }
        stage('Wait for Stack Completion') {
            steps {
                script {
                    // Wait for the CloudFormation stack creation to complete
                    bat '''
                        powershell -Command "aws cloudformation wait stack-create-complete --stack-name MyEC2InstanceStack"
                    '''
                }
            }
        }
        stage('Get EC2 Public IP') {
            steps {
                script {
                    // Retrieve the public IP of the created EC2 instance
                    def ec2InstanceDetails = bat(script: '''
                        powershell -Command "(aws ec2 describe-instances --filters Name=tag:Name,Values=MyEC2Instance --query Reservations[*].Instances[*].PublicIpAddress --output text)"
                    ''', returnStdout: true).trim()
                    echo "EC2 Public IP: ${ec2InstanceDetails}"
                    
                    // Store the EC2 public IP in an environment variable for Ansible
                    env.EC2_PUBLIC_IP = ec2InstanceDetails
                }
            }
        }
        stage('Run Ansible Playbook') {
            steps {
                script {
                    // Use Ansible to configure the EC2 instance via SSH (Ansible playbook is in the Git repo)
                    bat '''
                        ansible-playbook -i "${EC2_PUBLIC_IP},"  ansible_provision.yaml
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning up...'
        }
        success {
            echo 'EC2 Instance successfully created and configured via Ansible!'
        }
        failure {
            echo 'Failed to create or configure EC2 instance.'
        }
    }
}
