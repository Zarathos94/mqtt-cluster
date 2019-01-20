pipeline {
    agent {
        node {
            label "master"
        }
    }

    options {
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }

    environment {
        BRANCH_NAME = "master"
        SLACK_CHANNEL = "#realtime-builds"
        SERVICE_NAME = "naga/social/emqtt"
        ECR_REPO_URL = "${ECR_BASE_URL}/${SERVICE_NAME}"
        MSG_PREFIX = "*[${SERVICE_NAME}][${BRANCH_NAME}][${BUILD_NUMBER}]*"
        COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
        SHORT_COMMIT = "${COMMIT}".take(8)
    }

    stages {
        stage('Start') {
            steps {
            slackSend message: "${MSG_PREFIX}: Pipeline starting... ",
                color: "good",
                channel: "${SLACK_CHANNEL}",
                teamDomain: "${env.SLACK_TEAM_DOMAIN}",
                tokenCredentialId: "${env.SLACK_CREDENTIALS}"
            }
        }

        stage('Checkout') {
            steps {
                step([$class: 'WsCleanup'])
                checkout scm
            }
        }

        stage('Build image') {
            steps {
                ansiColor('xterm') {
                    retry(2) {
                        sh 'docker build --no-cache -t ${ECR_REPO_URL}:${BRANCH_NAME}-${BUILD_NUMBER} -t ${ECR_REPO_URL}:${BRANCH_NAME}-latest .'
                    }
                }
            }
        }

        stage('Docker push') {
            steps {
                sh 'rm  ~/.dockercfg || true'
                sh 'rm ~/.docker/config.json || true'

                withDockerRegistry([credentialsId:"${ECR_CREDENTIALS}", url:"https://${ECR_BASE_URL}"]) {
                    retry(2) {
                        sh 'docker push ${ECR_REPO_URL}:${BRANCH_NAME}-${BUILD_NUMBER}'
                        sh 'docker push ${ECR_REPO_URL}:${BRANCH_NAME}-latest'
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs deleteDirs: true, notFailBuild: true
        }
        success {
            slackSend message: "${MSG_PREFIX}: Pipeline finished. | ${env.BUILD_URL}",
                color: "good",
                channel: "${SLACK_CHANNEL}",
                teamDomain: "${env.SLACK_TEAM_DOMAIN}",
                tokenCredentialId: "${env.SLACK_CREDENTIALS}"

            
        }
        failure {
            slackSend message: "${MSG_PREFIX}: Pipeline failed. | ${env.BUILD_URL}",
                color: "danger",
                channel: "${SLACK_CHANNEL}",
                teamDomain: "${env.SLACK_TEAM_DOMAIN}",
                tokenCredentialId: "${env.SLACK_CREDENTIALS}"
        }
    }
}
