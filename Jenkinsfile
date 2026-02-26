pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = "learnjenkinsapp"
        AWS_DEFAULT_REGION = "eu-north-1"
        AWS_DOCKER_REGISTRY = "706108320219.dkr.ecr.eu-north-1.amazonaws.com"
        AWS_ECS_CLUSTER = "learn-jenkins-app-prod"
        AWS_ECS_SERVICE = "LearnJenkinsApp-TaskDefinition-Prod-service-kjb5zeeh"
        AWS_ECS_TD = "LearnJenkinsApp-TaskDefinition-Prod"
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

        stage('Build Docker image') {
                agent {
                    docker {
                        image 'my-aws-cli'
                        args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                        reuseNode true
                    }
                }
                steps {
                    withCredentials([usernamePassword(credentialsId: 'learning-s3', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {

                        sh '''
                            docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                            aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                            docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                        '''
                    }
                }
            }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
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
                        TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TD
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
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
