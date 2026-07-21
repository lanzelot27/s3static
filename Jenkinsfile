pipeline {
    agent any

    environment {
        AZ_CREDS = credentials('azure-storage-key')
        EC2_IP_1 = '10.0.142.82'
        EC2_IP_2 = '10.0.152.186'
        ACI_ENDPOINT = 'lanzy-nginx-app.centralindia.azurecontainer.io'
        STORAGE_ACCOUNT = 'lanzstrue'
        SHARE_NAME = 'webcontent'
    }

    stages {
        stage('Deploy to Staging (Azure ACI)') {
            steps {
                sh '''
                    az storage file upload \
                        --account-name $STORAGE_ACCOUNT \
                        --account-key $AZ_CREDS \
                        --share-name $SHARE_NAME \
                        --source index.html
                '''
            }
        }

        stage('Test Staging') {
            steps {
                sh '''
                    sleep 10
                    HTTP_CODE=$(curl -s -o /tmp/resp.txt -w "%{http_code}" http://$ACI_ENDPOINT)
                    if [ "$HTTP_CODE" -ne 200 ]; then
                        echo "Staging check failed with HTTP $HTTP_CODE"
                        exit 1
                    fi
                    grep -q "Hello" /tmp/resp.txt || { echo "Content assertion failed"; exit 1; }
                '''
            }
        }

        stage('Deploy to Production (AWS EC2)') {
            steps {
                sshagent(['aws-ec2-ssh-key']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no index.html ec2-user@$EC2_IP_1:/tmp/index.html
                        ssh -o StrictHostKeyChecking=no ec2-user@$EC2_IP_1 'sudo cp /tmp/index.html /usr/share/nginx/html/'

                        scp -o StrictHostKeyChecking=no index.html ec2-user@$EC2_IP_2:/tmp/index.html
                        ssh -o StrictHostKeyChecking=no ec2-user@$EC2_IP_2 'sudo cp /tmp/index.html /usr/share/nginx/html/'
                    '''
                }
            }
        }
    }
}
