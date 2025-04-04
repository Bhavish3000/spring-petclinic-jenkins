pipeline {
    agent {
        label 'Kubernetes'
    }

    triggers {
        cron('0 23 * * 1-5')
    }

    parameters {
        string(name: 'giturl', defaultValue: 'https://github.com/Bhavish3000/spring-petclinic.git', description: 'Git repo URL')
        string(name: 'gitbranch', defaultValue: 'deploy', description: 'Git repo branch')
        string(name: 'gitcredentials', defaultValue: 'GithubCredentials', description: 'GitHub login Credentials')
        string(name: 'kubernetesConfigCredential', defaultValue: 'kubernetes-config', description: 'Kubernetes config credentials ID')
        string(name: 'helmReleaseName', defaultValue: 'spcjenkins', description: 'Helm release name')
        string(name: 'helmNamespace', defaultValue: 'default', description: 'Helm namespace')
        string(name: 'helmChartPath', defaultValue: './k8s/spc', description: 'Helm chart path')
        string(name: 'dockerImage', defaultValue: 'bhavishtumalapenta/spring-petclinic-jenkins:345', description: 'Docker image for Helm upgrade')
        string(name: 'agentlabel', defaultValue: 'Kubernetes', description: 'Value of agents label')
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git url: params.giturl,
                    branch: params.gitbranch,
                    credentialsId: params.gitcredentials
            }
        }

        stage('Helm Install') {
            steps {
                script {
                    withCredentials([file(credentialsId: params.kubernetesConfigCredential, variable: 'KUBECONFIG')]) {
                        def releaseExists = sh(script: "helm list -n ${params.helmNamespace} | grep ${params.helmReleaseName}", returnStatus: true) == 0

                        if (!releaseExists) {
                            echo "Installing Helm release...."
                            sh """ 
                                helm install ${params.helmReleaseName} ${params.helmChartPath} --namespace ${params.helmNamespace}
                            """
                        }

                        else {
                            echo "Helm release already exists."
                        }
                    }
                }
            }
        }

        stage('Helm Upgrade') {
            steps {
                withCredentials([file(credentialsId: params.kubernetesConfigCredential, variable: 'KUBECONFIG')]) {
                    echo "Upgrading Helm release..."
                    sh """
                        helm upgrade ${params.helmReleaseName} ${params.helmChartPath}  \
                        --namespace ${params.helmNamespace} \
                        --set image=${params.dockerImage}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}
