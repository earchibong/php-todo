pipeline {
  agent any

      environment 
    {
        PROJECT     = 'php-todo'
        ECRURL      = '350100602815.dkr.ecr.eu-west-2.amazonaws.com/php-todo'
        DEPLOY_TO = 'develop'
    }

  stages {

    stage("Initial cleanup") {
        steps {
        dir("${WORKSPACE}") {
            deleteDir()
        }
        }
    }

    stage('Checkout')
    {
      steps {
      checkout scmGit(
        branches: [[name: '*/develop'], [name: '*/feature']], 
        extensions: [], 
        userRemoteConfigs: [[credentialsId: '23ef1a81-ff88-4724-9462-8134b6d8ad86', url: 'https://github.com/earchibong/php-todo.git']]
        )
      }
        }

    stage('Build preparations')
      {
        steps
          {
              script 
                {
                    // calculate GIT lastest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(7)
                    // calculate a sample version tag
                    VERSION = shortCommitHash
                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                }
            }
      }   

    stage('Build For Dev Environment') {
               when { branch pattern: "^feature.*|^bug.*|^dev", comparator: "REGEXP"}
            
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')") 
                sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("dev-$BUILD_NUMBER")
            }
            }
        }
      }

    stage('Build For Staging Environment') {
            when {
                expression { BRANCH_NAME ==~ /(staging|develop)/ }
            }
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')") 
                sh "docker build --network=host -t $IMAGE ."
                //docker.withRegistry("https://$ECRURL"){
                //docker.image("$IMAGE").push("dev-staging-$BUILD_NUMBER")
                }
            }
        }

    stage('Test For Staging Environment') {
        when {
            expression { BRANCH_NAME ==~ /(staging|develop)/}
        }

        steps {
            final String url = "http://localhost:8085"

                    withCredentials([usernameColonPassword(credentialsId: "jenkins-api-token", variable: "API_TOKEN")]) {
                        final def (String response, int code) =
                            sh(script: "curl -s -w '\\n%{response_code}' -u $API_TOKEN $url", returnStdout: true)
                                .trim()
                                .tokenize("\n")

                        echo "HTTP response status code: $code"

                        if (code == 200) {
                            echo response
            }

        }
    }

    stage('Deploy For Staging Environment'){
        when {
            expression { bBRANCH_NAME ==~ /(staging|develop)/}
        }

        steps{
            script{
              sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')")
              docker.withRegistry("https://$ECRURL"){
              docker.image("$IMAGE").push("dev-staging-$BUILD_NUMBER")  
              }
            }

        }
    }


    stage('Build For Production Environment') {
        when { tag "release-*" }
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')") 
                // sh "docker build --network=host -t $IMAGE -f deploy/docker/Dockerfile ."
                sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("prod-$BUILD_NUMBER")
                }
            }
        }
    }
  }

        post
    {
        always
        {
            sh "docker rmi -f $IMAGE "
        }
    }
} 