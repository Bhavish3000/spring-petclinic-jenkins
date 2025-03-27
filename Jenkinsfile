pipeline {
    agent any

    triggers {
        pollSCM('H/15 * * * *')
    }

    tools {
        maven 'maven3.9.9'
        jdk 'java17'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/Bhavish3000/spring-petclinic.git',
                    branch: 'main',
                    credentialsId: 'GithubCredentials'
            }
        }

        stage('Build') {
            steps {
                withMaven(
                    maven: 'maven3.9.9',
                    jdk: 'java17',
                    traceability: true
                ) {
                    sh 'mvn clean install'
                    junit testResults: '**/surefire-reports/*.xml'
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar',
                    fingerprint: true,
                    onlyIfSuccessful: true
            }
        }

        stage('SonarCloud analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'SONARCLOUD_TOKEN', installationName: 'SONAR_CLOUD') {
                    sh '''
                    mvn clean package \
                    org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar \
                    -Dsonar.organization=gameoflifebhavish \
                    -Dsonar.projectKey=1df882c96abe130b80ff99bc2fc7e4b745535a7f
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }


        stage('Upload to S3') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'ap-south-1') {
                    s3Upload(bucket: 'springpetclinicbhavishartifact', 
                             file: 'target/spring-petclinic-3.4.0-SNAPSHOT.jar', 
                             path: 'artifacts/')
                }
            }
        }
    }

    post {
        success {
            echo 'Build and artifact archiving completed successfully!'
        }

        failure {
            echo 'Build or artifact archiving failed!'
        }
    }
}
