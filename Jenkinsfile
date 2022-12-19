#!/groovy
pipeline {
    options {
        skipDefaultCheckout true
    }
    environment {
        APP_NAME = 'BuyRequest/sync-service'
        WORKSPACE = '/home/Jenkins/workspace/BuyRequest/sync-service'
    }
    agent none
    triggers {
        cron('H 23 * * *')
    }
    stages {
        stage('Preflight check'){
            agent {node {label 'master' }}
            when {
                branch 'dev'
            }
            post {
                success {
                    slackSend channel: 'jenkins-buyrequest', \
                    teamDomain: 'umbrellaitcom', \
                    color: '#5cb589', \
                    message: "${env.APP_NAME} - start update- ${env.JOB_BASE_NAME} ${env.BUILD_DISPLAY_NAME} Started (<${env.RUN_DISPLAY_URL}|Open>)",
                    tokenCredentialId: 'umbrella.devops-slack-integration-token'
                }
            }
            steps {
                sh "env | sort"
            }
        }

        stage('Deliver to PROD env'){
            agent {node {label 'master'}}
            when {
                branch 'master'
            }
            steps {
                script {
                    sshagent (credentials: ['buyrequestprod']) {
                        sh 'ssh -o StrictHostKeyChecking=no -l ubuntu buyrequest-prod.umbrellait.tech \
                        "cd /home/ubuntu/buy-request/services/sync-service \
                        && git checkout -f && git pull \
                        && sh ./scripts/sync-proto"'
                    }
                }
            }
        }
    }
    post {
        success {
            slackSend channel: 'jenkins-buyrequest', \
            teamDomain: 'umbrellaitcom', \
            color: '#5cb589', \
            message: "${env.APP_NAME} - ${env.JOB_BASE_NAME} ${env.BUILD_DISPLAY_NAME} Success! (<${env.RUN_DISPLAY_URL}|Open>)", \
            tokenCredentialId: 'umbrella.devops-slack-integration-token'
        }
        failure {
            slackSend channel: 'jenkins-buyrequest', \
            teamDomain: 'umbrellaitcom', \
            color: '#951d13', \
            message: "${env.APP_NAME} - ${env.JOB_BASE_NAME} ${env.BUILD_DISPLAY_NAME} Failed! (<${env.RUN_DISPLAY_URL}|Open>)", \
            tokenCredentialId: 'umbrella.devops-slack-integration-token'
        }
    }
}
