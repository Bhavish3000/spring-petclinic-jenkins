pipeline {
    agent none

    triggers {
        cron('0 18 * * 1-5')
    }

    environment {
        registry = 'bhavishtumalapenta/spring-petclinic-jenkins'
        registryCredential = 'dockerhub'
        dockerImage = ''
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/Bhavish3000/spring-petclinic.git',
                    branch: 'deploy',
                    credentialsId: 'GithubCredentials'
            }
        }

        stage('Infrastructure') {
            agent {
                label 'terraform'
            }
            steps {
                sh 'chmod +x ./Terraform.sh'

                sh './Terraform.sh'
            }
        }

        stage('Build Docker Image') {
            agent {
                label 'docker'
            }
            steps {
                script {
                    dockerImage = docker.build("${registry}:${BUILD_NUMBER}")

                }
            }
        }

        stage('Push to Docker Hub') {
            agent {
                label 'docker'
            }
            steps {
                script {
                    withCredentials([string(credentialsId: registryCredential, variable: 'DOCKER_PAT')]) {
                        sh """
                        echo $DOCKER_PAT | docker login --username bhavishtumalapenta --password-stdin
                        """
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Clean Up') {
            agent {
                label 'docker'
            }
            steps {
                script {
                    sh "docker rmi ${registry}:${BUILD_NUMBER}"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline for Infrastructure creation $ Docker iamge Build and Push completed successfully.'
        }

        failure {
            echo 'Pipeline for Infrastructure creation or Docker iamge Build and Push failed.'
        }
    }

}