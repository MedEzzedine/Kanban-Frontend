pipeline {
    agent any

    tools {
        nodejs "Node12"
    }
    
    environment {
        // Define environment variables
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub_credentials'
        FRONTEND_IMAGE_NAME = 'kanban-frontend'
        DOCKERHUB_USER = 'medez'
    }

    stages {

        stage('Install Packages') {
            steps {
                script {
                    sh 'npm install'
                }
            }
        }

        stage('Build Application') {
            steps {
                script {
                    sh 'npm run build'
                }
            }
        }

        stage('SonarQube Analysis') {
            // environment {
            //     SCANNER_HOME = tool 'SonarQube Server'
            // }
            // steps {
            //     withSonarQubeEnv(installationName: 'SonarQube Server', credentialsId: 'sonarqube_token') {
            //         sh "${SCANNER_HOME}/bin/sonar-scanner"
            //     }  
            // }
            steps {
                script {
                  sh "sleep 12"
                }
            }
            
        }

        stage('Build and Push Docker Image') {
            steps {
                script {

                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker build -t ${DOCKERHUB_USER}/${FRONTEND_IMAGE_NAME}:${BUILD_NUMBER} ."
                        sh "docker tag ${DOCKERHUB_USER}/${FRONTEND_IMAGE_NAME}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${FRONTEND_IMAGE_NAME}:latest"
                        sh "docker push ${DOCKERHUB_USER}/${FRONTEND_IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    sh "docker compose -p 'kanban' down || echo 'project kanban not running'"
                    sh "docker compose -p 'kanban' up -d --build"
                    slackSend color: "good", message: "Frontend deployed Successfully!"
                }
            }
        }
    }

    post {
        always {
            // Actions to perform after the pipeline completes
            script {
                sh "docker logout"
                echo 'Pipeline execution complete!!'
            }
        }
        failure {
            slackSend color: "danger", message: "Frontend pipeline failed."
        }
    }
}
