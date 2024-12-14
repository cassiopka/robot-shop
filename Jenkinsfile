pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'f86b8146-c934-4d88-a298-b0e675dd9be6'
    }

    stages {
        stage('Checkout on TEST') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'test']],
                        userRemoteConfigs: [[url: 'https://github.com/cassiopka/robot-shop.git']]
                    ])
                    env.BRANCH_NAME = 'test'
                }
            }
        }

        stage('Checkout on DEV') {
            agent {
                node {
                    label 'dev'
                }
            }
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'dev']],
                        userRemoteConfigs: [[url: 'https://github.com/cassiopka/robot-shop.git']]
                    ])
                    env.BRANCH_NAME = 'dev'
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { env.BRANCH_NAME == 'test' || env.BRANCH_NAME == 'dev' }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    }

                    sh 'docker-compose build'
                }
            }
        }

        stage('Push Docker Image') {
            when {
                expression { env.BRANCH_NAME == 'test' || env.BRANCH_NAME == 'dev' }
            }
            steps {
                script {
                    sh 'docker-compose push'
                }
            }
        }

        stage('Deploy to DEV') {
            agent {
                node {
                    label 'dev'
                }
            }
            when {
                expression { env.BRANCH_NAME == 'dev' }
            }
            steps {
                script {
                    sh 'docker-compose pull'
                    sh 'docker-compose up -d'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
