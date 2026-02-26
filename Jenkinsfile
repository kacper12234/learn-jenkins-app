pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = "eu-north-1"
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            environment {
                BUCKET = 'learn-jenkins-kacper'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'learning-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                        aws ecs update-service --cluster learn-jenkins-app-prod --service LearnJenkinsApp-TaskDefinition-Prod-service-kjb5zeeh --task-definition LearnJenkinsApp-TaskDefinition-Prod
                    '''
                }
            }
        }



        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production', ok: 'Yes, I am sure!'
                }
            }
        }


    }
}
