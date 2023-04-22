pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
      environment {
       ECR_URI = '055745520549.dkr.ecr.us-east-1.amazonaws.com/app_pipeline'
    }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/labanoelie/geolocation.git'
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
          // Building Docker images
       // stage('Building image') {
        //    steps{
         //       script {
        //            dockerImage = docker.build registry + ":$BUILD_NUMBER"
        //        }
       //     }
      //  }
        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
                    sh "docker build -t $ECR_URI:latest ."
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_URI"
                    sh "docker push $ECR_URI:latest"

                }
            }
        }
        //deploy the image that is in ECR to our EKS cluster
        stage ("Kube Deploy") {
            steps {
                withKubeConfig([credentialsId: 'eks_credential', serverUrl: '']) {
                 sh "kubectl apply -f eks_deploy_from_ecr.yaml"
                }
            }
        }
    }
}
