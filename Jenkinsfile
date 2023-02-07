  
pipeline {
  agent any

      environment 
    {
        AWS_ACCOUNT_ID = ' 350100602815'
        AWS_DEFAULT_REGION = 'eu-west-2'
        IMAGE_REPO_NAME = 'php-todo'
        IMAGE_TAG = 'latest'
        REPOSITORY_URI = '${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}'
    }

  stages {

    stage("Initial cleanup") {
        steps {
        dir("${WORKSPACE}") {
            deleteDir()
        }
        }
    }

    stage('Logging into AWS ECR'){
      steps {
        script {
            //sh ("eval \$(aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com)")
            sh(label: 'AWS ECR login', script:
               '''
               #!/bin/bash
               eval $(aws ecr get-login-password —-region ${AWS_DEFAULT_REGION} | docker login —-username AWS —-password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com)
               '''.stripIndent())
        }
      }
    }
    
    stage('Checkout SCM'){
      steps {
        checkout([
        $class: 'GitSCM', 
        doGenerateSubmoduleConfigurations: false, 
        extensions: [],
        submoduleCfg: [], 
        branches: [[name: 'main']],
        userRemoteConfigs: [[url: "https://github.com/earchibong/php-todo.git ",credentialsId:'']] 	
        ])  
      }
    }

    stage('Build Image') {
        steps {
            echo 'Build Dockerfile....'
            script {
                dockerImage = docker.build '${IMAGE_REPO_NAME}:${IMAGE_TAG}'
                //sh "docker build --network=host -t $IMAGE_REPO_NAME:$IMAGE_TAG ."
            }
        }
    }
  }
}