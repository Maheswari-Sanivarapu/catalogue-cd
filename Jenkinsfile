pipeline {
    agent{
        label 'AGENT_1'
    }
    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = "315069654700"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
        CLUSTER_NAME = "my-cluster"
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
                        // here updating the kube-config like an authentication and getting the nodes,creating the namespace
                        // here sed editor we are adding means appVersion we got from CI using the appVersion to CD as downstream so this will pass the appVersion using sed-editor in values-dev.yaml and CD will trigger with that version
                        // deploying the application to kubernetes using helm 
                        sh """
                            aws eks update-kubeconfig --region $REGION --name $CLUSTER_NAME
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml 
                            helm upgrade --install $COMPONENT values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }
    }
}