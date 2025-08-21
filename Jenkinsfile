pipeline {
    agent{
        label 'AGENT_1'
    }
    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = "315069654700"
        CLUSTER_NAME = 'my-cluster'
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters{
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    stages{
        stage('Deploy'){
            steps{
                script{
                    withAWS(credentials: 'aws-creds', region: 'us-east-1'){
                        sh """
                            aws eks update-kubeconfig --region $REGION --name $CLUSTER_NAME
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -n $PROJECT .
                        """
                    }
                }
            }
        }
    }
}