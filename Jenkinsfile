pipeline {
    agent none

    triggers {
        cron('0 18 * * 1-5')
    }

    parameters {
        string(name: 'giturl', defaultValue: 'https://github.com/Bhavish3000/spring-petclinic.git', description: 'Git repo URL')
        string(name: 'gitbranch', defaultValue: 'deploy', description: 'Git repo branch')
        string(name: 'gitcredentials', defaultValue: 'GithubCredentials', description: 'GitHub login Credentials')
        string(name: 'InfaagentLabel', defaultValue: 'terraform', description: 'Lebel for the Infra agent')
        string(name: 'Terraformshellscript', defaultValue: './Terraform.sh', description: 'Terraforms configuration files execution Shell script')
        string(name: 'registry', defaultValue: 'bhavishtumalapenta/spring-petclinic-jenkins', description: 'Docker registry')
        string(name: 'registryCredential', defaultValue: 'dockerhub', description: 'Docker Hub credentials ID')
        string(name: 'DockerhubUsername', defaultValue: 'bhavishtumalapenta', description: 'User name for Dockerhub')
        string(name: 'DOckeragentLabel', defaultValue: 'docker', description: 'Lebel for the Docker agent')

    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: params.giturl,
                    branch: params.gitbranch,
                    credentialsId: params.gitcredentials
            }
        }

        stage('Infrastructure') {
            agent {
                label params.InfaagentLabel
            }
            steps {
                sh 'chmod +x ${params.Terraformshellscript}'

                sh '${params.Terraformshellscript}'
            }
        }

        stage('Build Docker Image') {
            agent {
                label params.DOckeragentLabel
            }
            steps {
                script {
                    dockerImage = docker.build("${params.registry}:${BUILD_NUMBER}")

                }
            }
        }

        stage('Push to Docker Hub') {
            agent {
                label params.DOckeragentLabel
            }
            steps {
                script {
                    withCredentials([string(credentialsId: params.registryCredential, variable: 'DOCKER_PAT')]) {
                        sh """
                        echo $DOCKER_PAT | docker login --username ${params.DockerhubUsername} --password-stdin
                        """
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Clean Up') {
            agent {
                label params.DOckeragentLabel
            }
            steps {
                script {
                    sh "docker rmi ${params.registry}:${BUILD_NUMBER}"
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