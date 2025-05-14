pipeline {
    agent any

    environment {
        NODE_ENV = ''
        EC2_USER = 'ubuntu'
        REMOTE_PATH = '/home/ubuntu/Prueba-de-despliege'
        SSH_KEY = credentials('ssh-key-ec2')
    }

    stages {
        stage('Setup Environment Variables') {
            steps {
                script {
                    def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    env.BRANCH_NAME = branch

                    if (branch == 'main') {
                        env.NODE_ENV = 'production'
                        env.EC2_IP = '54.163.72.1'
                    } else if (branch == 'qa') {
                        env.NODE_ENV = 'qa'
                        env.EC2_IP = '3.224.80.215'
                    } else if (branch == 'develop') {
                        env.NODE_ENV = 'development'
                        env.EC2_IP = '18.234.56.3'
                    } else {
                        error "Branch ${branch} not configured for deployment."
                    }
                }
            }
        }

        // stage('Checkout') {
        //     steps {
        //         git branch: "${env.BRANCH_NAME}", url: 'https://github.com/daviddlcm/Prueba-de-despliege'
        //     }
        // }

        stage('Build') {
            steps {
                sh 'rm -rf node_modules'
                sh 'npm ci'
            }
        }

        stage('Deploy') {
            steps {
                sh """
                ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$EC2_IP '
                    cd $REMOTE_PATH &&
                    git pull origin ${env.BRANCH_NAME} &&
                    npm ci &&
                    pm2 restart health-api-${env.NODE_ENV} || pm2 start server.js --name health-api-${env.NODE_ENV}
                '
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment to ${env.NODE_ENV} successful!"
        }
        failure {
            echo "❌ Deployment to ${env.NODE_ENV} failed!"
        }
    }
}
