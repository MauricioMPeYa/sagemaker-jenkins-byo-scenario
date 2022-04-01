pipeline {

    agent any

    environment {
        AWS_ECR_LOGIN = 'true'
        DOCKER_CONFIG= "${params.JENKINSHOME}"
    }

    stages {
        stage("Checkout") {
            steps {
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/MauricioMPeYa/sagemaker-jenkins-byo-scenario']]])
            }
        }

        stage("BuildPushContainer") {
            steps {
              sh """
                echo "${params.ECRURI}"
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${params.ECRURI}
 	              docker build -t scikit-byo:${env.BUILD_ID} .
                docker tag scikit-byo:${env.BUILD_ID} ${params.ECRURI}:${env.BUILD_ID} 
                docker push ${params.ECRURI}:${env.BUILD_ID}
              """
            }
        }

      stage("DeployToTest") {
            steps { 
              sh """
               aws sagemaker create-model --model-name ${params.SAGEMAKER_TRAINING_JOB}-${env.BUILD_ID}-Test --primary-container ContainerHostname=${env.BUILD_ID},Image=${params.ECRURI}:${env.BUILD_ID},ModelDataUrl='${S3_MODEL_ARTIFACTS}'/${params.SAGEMAKER_TRAINING_JOB}-16/output/model.tar.gz,Mode='SingleModel' --execution-role-arn ${params.SAGEMAKER_EXECUTION_ROLE_TEST}
               aws sagemaker create-endpoint-config --endpoint-config-name ${params.SAGEMAKER_TRAINING_JOB}-${env.BUILD_ID}-Test --production-variants VariantName='single-model',ModelName=${params.SAGEMAKER_TRAINING_JOB}-${env.BUILD_ID}-Test,InstanceType='ml.m4.xlarge',InitialVariantWeight=1,InitialInstanceCount=1
               aws sagemaker create-endpoint --endpoint-name ${params.SAGEMAKER_TRAINING_JOB}-${env.BUILD_ID}-Test --endpoint-config-name ${params.SAGEMAKER_TRAINING_JOB}-${env.BUILD_ID}-Test
               sleep 300
              """
             }
        }

  }
}   
