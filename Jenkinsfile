pipeline {
    agent any
    environment {
        registry = "024193401724.dkr.ecr.us-east-1.amazonaws.com/sample"
    }
   
     stages {
        stage('Cloning Git') {
            steps {
                 // checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/akannan1087/myPythonDockerRepo']]])     
                git branch: 'master', credentialsId: 'git-creds', url: 'https://aiyubalikhan9295@bitbucket.org/aiyubalikhan92/hippo.git'
            }
        }
  
  
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry
        }
      }
    }
   
    // Uploading Docker images into AWS ECR

 
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 024193401724.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker push 024193401724.dkr.ecr.us-east-1.amazonaws.com/sample:latest'
         }
        }
      }
   
         // Stopping Docker containers for cleaner Docker run
     stage('stop previous containers') {
         steps {
            sh 'docker ps -f name=mypythonContainer -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=mypythonContainer -q | xargs -r docker container rm'
         }
       }
      
    stage('Docker Run') {
     steps{
         script {
                sh 'docker run -d -p 8096:5000 --rm --name mypythonContainer 024193401724.dkr.ecr.us-east-1.amazonaws.com/sample:latest'
            }
      }
    }
     
    stage('zip creation'){
        steps{
         script {

       sh 'zip -r Dockerrun.aws.json.zip . -x "/var/lib/jenkins/workspace/hippo/Dockerrun.aws.json"'

       archiveArtifacts artifacts: 'Dockerrun.aws.json.zip', fingerprint: false

         }
        }
    }

    stage(' cp zip '){
        steps{
         script {

       sh 'aws s3 cp Dockerrun.aws.json.zip s3://liraan-test/vb-automation/ --content-type application/zip'
         }
        }
    }

       


    stage(' create-application-version '){
        steps{
         script {

      sh 'aws elasticbeanstalk create-application-version --application-name python --version-label versions$BUILD_NUMBER --source-bundle S3Bucket=liraan-test,S3Key="Dockerrun.aws.json.zip" --region us-east-1'

         }
        }
    }

    stage(' update-environment '){
         steps{
         script {

      sh 'aws elasticbeanstalk update-environment --application-name Python --environment-name Python-env-1 --version-label versions$BUILD_NUMBER --region us-east-1'
}
}
      }
     
    
    }
}

