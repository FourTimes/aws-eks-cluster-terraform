pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
        REGION = credentials('REGION')
        ECR_REPO = credentials('ECR_REPO')
        ECR_REPO_URL = credentials('ECR_REPO_URL')
        EKS_CLUTER_NAME = "tspring-staging"
        GIT_COMMIT_ID = sh(script: 'git rev-parse --short HEAD', returnStdout: true)
        IMAGE_NAME = "api-${GIT_COMMIT_ID}"
//         SONAR_PROJECT_NAME= "tspring-api"
//         SONAR_PROJECT_LOGIN = credentials('tspring-api-login')
    }
    stages {
//         stage('SONAR CODE QUALITY') {
//         steps {
//             sh '''
//                 sonar-scanner \
//                     -Dsonar.projectKey=${SONAR_PROJECT_NAME} \
//                     -Dsonar.sources=. \
//                     -Dsonar.host.url=https://sonar.tspring.co \
//                     -Dsonar.login=${SONAR_PROJECT_LOGIN}
//                 '''
//             }
//         }
        stage('DOCKER IMAGE BUILD') {
        steps {
            sh '''
                docker build -t $ECR_REPO:${IMAGE_NAME} .
                echo "completed"
                echo $IMAGE_NAME
                '''
            }
        }
        stage('DOCKER IMAGE PUSH') {
        steps {
            sh '''
                aws configure set region $REGION
                $(aws ecr get-login --region $REGION --no-include-email)
                docker push $ECR_REPO:$IMAGE_NAME
                docker logout $ECR_REPO_URL
                docker rmi $ECR_REPO:$IMAGE_NAME
                echo "completed"
                '''
            }
        }
    stage('KUBERNETES DEPLOYMENT') {
        steps {
            sh '''
                export imageNameandversion=$ECR_REPO:$IMAGE_NAME
                sed -i "s|containerImageName|$imageNameandversion|" kube-deployment/api-server.yml
                aws eks --region $REGION update-kubeconfig --name $EKS_CLUTER_NAME
                /usr/local/bin/kubectl apply -f kube-deployment/ -n development
                unset imageNameandversion
            '''
            }
        }
    }
}
