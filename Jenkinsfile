pipeline {
    agent any

    triggers {
        cron('0 18 * * 1-5')
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
            steps {
                sh 'python3 terrform.py'
            }
        }