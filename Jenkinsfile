pipeline {
    agent {
        label params.agentlabel
    }

    triggers {
        pollSCM('H/15 * * * *')
    }

    parameters {
        string(name: 'agentlabel', defaultValue: 'Build', description: 'agent label')
        string(name: 'giturl', defaultValue: 'https://github.com/Bhavish3000/spring-petclinic.git', description: 'github repository url')
        string(name: 'gitbranch', defaultValue: 'main', description: "github repository branch")
        string(name: 'GithubCredentialsID', defaultValue: 'GithubCredentials', description: 'Github account Credentials')
        string(name: 'sonarcredentials', defaultValue: 'SONARCLOUD_TOKEN', description: 'Access token for the sonar cloud')
        string(name: 'sonarInstallationName', defaultValue: 'SONAR_CLOUD', description: 'Installation Name of Sonar cloud in system configuration')
        string(name: 'aws_credentials', defaultValue: 'aws-credentials', description: 'Accesskey and secreatekey of AWs iam user')
        string(name: 'aws_region', defaultValue: 'ap-south-1', description: 'AWS global region')
        string(name: 's3_bucket', defaultValue: 'springpetclinicbhavishartifact', description: 'Name of the AWS s3 bucket')



    }

    tools {
        maven 'maven3.9.9'
        jdk 'java17'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: params.giturl,
                    branch: params.gitbranch,
                    credentialsId: params.GithubCredentialsID
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
                withSonarQubeEnv(credentialsId: params.sonarcredentials, installationName: params.sonarInstallationName) {
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
                withAWS(credentials: params.aws_credentials, region: params.aws_region) {
                    s3Upload(bucket: params.s3_bucket, 
                             file: 'target/spring-petclinic-3.4.0-SNAPSHOT.jar', 
                             path: 'artifacts/')
                }
            }
        }
    }

    post {
        success {
            echo 'Build, artifact archiving and Static code analysis completed successfully!'
        }

        failure {
            echo 'Build or artifact archiving  or Static code analysis failed!'
        }
    }
}
