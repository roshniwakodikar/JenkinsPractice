pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1' // Adjust to your region
        CFN_STACK_NAME = 'EC2Stack' // Define the stack name here
    }
    stages {
        stage('Create EC2 Instance') {
            steps {
                script {
                    // Use the stack name in the CloudFormation create command
                    sh '''
                        aws cloudformation create-stack --stack-name $CFN_STACK_NAME --template-body file://cloudformation_ec2.yaml --region $AWS_REGION
                        aws cloudformation wait stack-create-complete --stack-name $CFN_STACK_NAME --region $AWS_REGION
                    '''
                }
            }
        }

        stage('Run Ansible Provisioning') {
            steps {
                script {
                    // Pass the stack name to the Ansible playbook
                    sh '''
                        ansible-playbook ansible_provision.yaml -e "stack_name=$CFN_STACK_NAME"
                    '''
                }
            }
        }
    }
}
