pipeline {
    agent none

    environment {
        DOCKER_CREDENTIALS_ID = 'f86b8146-c934-4d88-a298-b0e675dd9be6'
    }

    stages {
        stage('Checkout on PROD') {
            agent {
                node {
                    label 'prod'
                }
            }
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'prod']],
                        userRemoteConfigs: [[url: 'https://github.com/cassiopka/robot-shop.git']]
                    ])
                    env.BRANCH_NAME = 'prod'
                }
            }
        }

        stage('Deploy to PROD') {
            agent {
                node {
                    label 'prod'
                }
            }
            when {
                expression { env.BRANCH_NAME == 'prod' }
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
            node {
                label 'prod'
            }
            steps {
                cleanWs()
            }
        }
    }
}
