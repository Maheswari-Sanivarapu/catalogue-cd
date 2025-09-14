pipeline {
    agent{
        label 'LABEL_1'
    }
    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = 557690617909
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        //disableConcurrentBuilds()
    }
    parameters{
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    } 
    stages{
        stage('Deploy'){
                steps{
                    script{
                        withAWS(credentials: 'aws-auth', region: 'us-east-1'){
                            sh """
                                aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                                kubectl get nodes
                                kubectl apply -f 01-namespace.yaml
                                sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml 
                                helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                            """
                        }
                    }
                }
            }
        stage('check status'){
            steps{
                script{
                    withAWS(credentials: 'aws-creds', region: 'us-east-1'){
                        def deploymentStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/$COMPONENT --timeout=30s -n $PROJECT || echo FAILED").trim()
                        if (deploymentStatus.contains("successfully rolled out")){
                            echo "Deployment is Success"
                        } else {
                            sh """
                                helm rollback $COMPONENT -n $PROJECT
                            """
                            def rollbackStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/$COMPONENT --timeout=30s -n $PROJECT || echo FAILED").trim()
                            if (rollbackStatus.contains("successfully rolled out")){
                                error"Deployment is Failure, Rollback is success"
                            } else {
                                error "Deployment is Failure, Rollback is also Failure, Application is not running"
                            }
                        }
                    }
                }
            }
        }
    }
}