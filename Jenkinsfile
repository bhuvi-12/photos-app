node{
    def GITHUB_PROJECT_URL = "https://github.com/bhuvi-12/photos-app.git"
    def AWS_ACCOUNT_ID = "347476671573"
    def AWS_REGION = "us-east-1"
    def AWS_JENKINS_CREDENTIALS_ID = "aws-credentials"
    def AWS_ECR_IMAGE = "uat-bki-fe-test"
    def AWS_EKS_CLUSTER_NAME = "uat-bki-eks-cluster"
    def EKS_NAMESPACE = "uat-test"
    def EKS_DEPLOYMENT_FILE = "deployment.yaml"
    def EKS_DEPLOYMENT_NAME = "photosapplication-deployment"
    def RUNNING_CONTAINER_NAME = "photos-application"
    def BUILD_NUMBER = currentBuild.number
    def IMAGE_VERSION = "v${BUILD_NUMBER}"
    
    stage("Code checkout"){
        git branch: "master", url: "${GITHUB_PROJECT_URL}"
    }
    stage("Project build & Push to ECR"){
        docker.withRegistry(
            "https://${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com",
            "ecr:${AWS_REGION}:${AWS_JENKINS_CREDENTIALS_ID}") {
            def dockerImage = docker.build("${AWS_ECR_IMAGE}:${IMAGE_VERSION}")
            dockerImage.push()
        }
    }
    stage("Deployment to EKS cluster"){
        withAWS(credentials: "${AWS_JENKINS_CREDENTIALS_ID}", region: "${AWS_REGION}") {
            sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${AWS_EKS_CLUSTER_NAME}"
            sh "kubectl set image deployment.apps/${EKS_DEPLOYMENT_NAME} -n ${EKS_NAMESPACE} ${RUNNING_CONTAINER_NAME}=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${AWS_ECR_IMAGE}:latest"
            sh "kubectl rollout status deployment/${EKS_DEPLOYMENT_NAME} -n ${EKS_NAMESPACE}"
        }
    }
}
