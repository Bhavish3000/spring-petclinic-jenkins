pipeline {
    agent {
        label 'Kubernetes'
    }

    triggers {
        cron('0 23 * * 1-5')
    }

    environment {
        KUBECONFIG = credentials('kubernetes-config')
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/Bhavish3000/spring-petclinic.git',
                    branch: 'deploy',
                    credentialsId: 'GithubCredentials'
            }
        }
        

        stage('Helm Install') {
            steps {
                script {
                    def releaseExists = sh(script: "helm list -n default | grep spcjenkins", returnStatus: true) == 0

                    if (!releaseExists) {
                        echo "Installing Helm release...."
                        sh """ 
                            helm install spcjenkins ./k8s/spc --namespace default"""
                    }

                    else {
                        echo "Helm release already exists."
                    }
                }
            }
        }

        stage('Helm Upgrade') {
            steps {
                
                echo "Upgrading Helm release..."
                sh """
                    helm upgrade spcjenkins ./k8s/spc  \
                    --namespace default \
                    --set image=bhavishtumalapenta/spring-petclinic-jenkins:345
                """
            }
        }
    }

}