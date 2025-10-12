pipeline {
  agent any
          
  stages {
    stage('Git Checkout') {
      steps {
        echo 'This stage is to clone the repo from github'
        git branch: 'master', url: "https://github.com/ishu162002/star-agile-health-care.git"
                        }
            }
    stage('Create Package') {
      steps {
        echo 'This stage will compile, test, package my application'
        sh 'mvn package'
                          }
            }
    
    /* stage('Create Docker Image') {
      steps {
        echo 'This stage will Create a Docker image'
        sh 'docker build -t ishu162002/healthcare:1.0 .'
                          }
            }
     stage('Docker-Login') {
           steps {
              withCredentials([usernamePassword(credentialsId: 'dockercreds', passwordVariable: 'dockerpassword', usernameVariable: 'dockerlogin')]) {
               sh 'docker login -u ${dockerlogin} -p ${dockerpassword}'
                             
                        }
                }
}
    stage('Docker Push-Image') {
      steps {
        echo 'This stage will push my new image to the dockerhub'
        sh 'docker push ishu162002/healthcare:1.0'
            }
      } */
    stage('AWS-Login') {
  steps {
    withCredentials([usernamePassword(credentialsId: 'awslogin', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
      sh '''
        echo "Logged in to AWS CLI"
        aws sts get-caller-identity
      '''
    }
  }
}

    stage('setting the Kubernetes Cluster') {
      steps {
        dir('terraform_files'){
          sh 'terraform init'
          sh 'terraform validate'
          sh 'terraform apply --auto-approve'
          sh 'sleep 20'
        }
      }
    }
    stage('deploy kubernetes'){
steps{
  sh 'sudo chmod 600 ./terraform_files/sir.pem'    
  sh 'minikube start'
  sh 'sleep 30'
  sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/sir.pem deployment.yml ubuntu@172.31.17.230:/home/ubuntu/'
  sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/sir.pem service.yml ubuntu@172.31.17.230:/home/ubuntu/'
script{
  try{
  sh 'ssh -o StrictHostKeyChecking=no -i ./terraform_files/sir.pem ubuntu@172.31.17.230 kubectl apply -f .'
  }catch(error)
  {
  sh 'ssh -o StrictHostKeyChecking=no -i ./terraform_files/sir.pem ubuntu@172.31.17.230 kubectl apply -f .'
  }
}
}
}
  }
}

    
