pipeline {
    agent {
        docker {
            // TODO build & push your Jenkins agent image, place the URL here
            image '352708296901.dkr.ecr.eu-central-1.amazonaws.com/yf-bot-reg:latest'
            args  '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    // options used from - https://www.jenkins.io/doc/book/pipeline/syntax/
    options {
    buildDiscarder(logRotator(daysToKeepStr: '30'))
    disableConcurrentBuilds()
    timestamps()

    }
    // env and image vars
    environment {
    IMAGE_NAME = "YF_jenkinsBot"
    IMAGE_TAG = "0.0.$BUILD_NUMBER"
    WORKSPACE = "/var/lib/jenkins/workspace/BotBuild/services"
    ECR_REGISTRY = "352708296901.dkr.ecr.eu-central-1.amazonaws.com/yf-bot-reg"

    }

    stages {
        stage('Build') {
            steps {

                sh '''
                aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin $ECR_REGISTRY
                cd /home/ec2-user/workspace/dev/botBuild/services/bot/
                docker build -t $IMAGE_NAME:$IMAGE_TAG .

                '''
            }
        }
    // used snyk container test from - https://docs.snyk.io/snyk-cli/commands/container-test
    stage('Snyx Check') {
    steps {
            withCredentials([string(credentialsId: 'Snyx', variable: 'SNYK_TOKEN')]) {
                sh 'snyk container test $IMAGE_NAME:$IMAGE_TAG --severity-threshold=critical --file=/home/ec2-user/workspace/dev/botBuild/services/bot/Dockerfile'
            }
        }
    }

    stage('Continue_Build') {
        steps {
            sh'''
            docker tag $IMAGE_NAME:$IMAGE_TAG $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
            docker push $ECR_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
            '''
        }
        // post used from docker docs - https://docs.docker.com/engine/reference/commandline/image_prune/
        //The following removes images created more than 1 week ago for any case
        post {
            always {
            sh '''
            echo 'One way or another, I have finished'
            docker image prune -a --filter "until=168h"
            '''
            }
         success {
            echo 'I succeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }


        stage('Trigger Deploy') {
            steps {
                build job: 'botDeploy', wait: false, parameters: [
                    string(name: 'BOT_IMAGE_NAME', value: "${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}")
                ]
            }
        }
    }
}