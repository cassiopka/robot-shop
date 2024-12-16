pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'f86b8146-c934-4d88-a298-b0e675dd9be6'
        SONAR_CREDENTIALS_ID = 'ee167194-2d67-400e-b640-14cc1cbad27c'
    }
    

    stages {
        stage('Checkout on TEST') {
            agent {
                node {
                    label 'test'
                }
            }
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

        // stage('Checkout on DEV') {
        //     agent {
        //         node {
        //             label 'dev'
        //         }
        //     }
        //     steps {
        //         script {
        //             checkout([
        //                 $class: 'GitSCM',
        //                 branches: [[name: 'dev']],
        //                 userRemoteConfigs: [[url: 'https://github.com/cassiopka/robot-shop.git']]
        //             ])
        //             env.BRANCH_NAME = 'dev'
        //         }
        //     }
        // }


        stage('Code Unit testing') {
            agent {
                docker {
                    image 'golang:1.23.4'
                    args '-v /home/jenkins/agent/workspace:/workspace'
                }
            }
            when {
                expression { env.BRANCH_NAME == 'test' || env.BRANCH_NAME == 'dev' }
            }
            steps {
                script {
                    sh 'go mod init github.com/cassiopka/robot-shop.git/distplash'
                    sh 'go test -v tests/'
                }
            }
        }

        stage('Code Quality Analysis') {
            agent {
                node {
                    label 'test'
                }
            }
            when {
                expression { env.BRANCH_NAME == 'test' || env.BRANCH_NAME == 'dev' }
            }
            steps {
                script {
                        sh 'cd shipping && mvn checkstyle:checkstyle'
                }
            }
        }

        stage('Security Analysis') {
            agent {
                node {
                    label 'test'
                }
            }
            when {
                expression { env.BRANCH_NAME == 'test' || env.BRANCH_NAME == 'dev' }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: SONAR_CREDENTIALS_ID, usernameVariable: 'SONAR_USERNAME', passwordVariable: 'SONAR_PASSWORD')]) {
                            sh 'cd shipping && mvn clean compile && mvn sonar:sonar -Dsonar.login=$SONAR_USERNAME -Dsonar.password=$SONAR_PASSWORD'
                    }
                }
            }
        }
        stage('Build Docker Image') {
            agent {
                node {
                    label 'test'
                }
            }
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
            agent {
                node {
                    label 'test'
                }
            }
            when {
                expression { env.BRANCH_NAME == 'test' || env.BRANCH_NAME == 'dev' }
            }
            steps {
                script {
                    sh 'docker-compose push'
                }
            }
        }

        // stage('Deploy to DEV') {
        //     agent {
        //         node {
        //             label 'dev'
        //         }
        //     }
        //     when {
        //         expression { env.BRANCH_NAME == 'dev' }
        //     }
        //     steps {
        //         script {
        //             sh 'docker-compose pull'
        //             sh 'docker-compose up -d'
        //         }
        //     }
        // }
    }

    post {
        always {
            cleanWs()
        }
    }
}
