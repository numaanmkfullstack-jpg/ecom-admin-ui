pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    environment {
        DOCKERHUB_USER = 'iamnmk777'
        IMAGE_NAME     = 'ecom-admin-ui'
        FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set Environment') {
            steps {
                script {
                    def branch = env.BRANCH_NAME ?: env.GIT_BRANCH?.replace('origin/', '')
                    if (branch == 'main') {
                        env.DEPLOY_TAG = 'prod'
                        env.DEPLOY_ENV = 'production (prod APP VM)'
                    } else if (branch == 'stage') {
                        env.DEPLOY_TAG = 'staging'
                        env.DEPLOY_ENV = 'staging (preprod APP VM)'
                    } else {
                        error("Only 'stage' and 'main' branches deploy. Current branch: ${branch}")
                    }
                }
            }
        }

        stage('Approve Prod Deploy') {
            when {
                expression { env.DEPLOY_TAG == 'prod' }
            }
            steps {
                input message: "Deploy ${FULL_IMAGE}:prod to PRODUCTION?", ok: 'Approve'
            }
        }

        stage('Build & Push') {
            steps {
                script {
                    env.GIT_SHA = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                }
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker build -t ${FULL_IMAGE}:${DEPLOY_TAG} -t ${FULL_IMAGE}:${GIT_SHA} .
                        docker push ${FULL_IMAGE}:${DEPLOY_TAG}
                        docker push ${FULL_IMAGE}:${GIT_SHA}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pushed ${FULL_IMAGE}:${DEPLOY_TAG}. Argo CD Image Updater on ${DEPLOY_ENV} will roll out the new digest."
        }
    }
}
