pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'f86b8146-c934-4d88-a298-b0e675dd9be6'
    }

    stages {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    }

                    sh 'docker-compose build'

                    sh 'docker-compose push'
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
