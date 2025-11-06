pipeline {
    agent any
    
    environment {
        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"
        PROJECT_ID = 'arched-proton-477313-g2'
        IMAGE_NAME = 'healthcare'
        IMAGE_TAG = 'v1.0'
        REGION = 'us-central1'
        CLUSTER_NAME = 'medicure-test-cluster'
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo 'Cloning the Medicure microservice repo from GitHub...'
                git branch: 'master', url: 'https://github.com/ishu162002/star-agile-health-care.git'
            }
        }

        stage('Build and Package') {
            steps {
                echo 'Building the Maven project...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                dir("${WORKSPACE}") { 
                    sh "docker build -t ishupurwar/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'GCP_SERVICE_ACCOUNT_KEY', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh "gcloud config set project ${PROJECT_ID}"
                }
            }
        }

        stage('Do GCP stuff') {
            steps {
                sh 'gcloud compute instances list'
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                echo "Deploying application to GKE..."

                withCredentials([file(credentialsId: 'GCP_SERVICE_ACCOUNT_KEY', variable: 'GOOGLE_CRED')]) {
                    sh '''
                        echo "Activating GCP Service Account..."
                        gcloud auth activate-service-account --key-file=$GOOGLE_CRED
                        gcloud container clusters get-credentials medicure-test-cluster --region us-central1 --project arched-proton-477313-g2

                        echo "Deploying to Kubernetes..."
                        kubectl apply -f ~/star-agile-health-care/deployment.yml
                        kubectl apply -f ~/star-agile-health-care/service.yml
                    '''
                }
            }
        }

        stage('Post Deployment Verification') {
            steps {
                echo 'Verifying Kubernetes Deployment...'
                sh '''
                    kubectl get pods -o wide
                    kubectl get svc
                '''
            }
        }
    }  // ✅ this closes the stages block

    post {
        success {
            echo '✅ Deployment Successful on GCP Kubernetes Cluster!'
        }
        failure {
            echo '❌ Pipeline Failed. Check logs for errors.'
        }
    }
}  // ✅ this closes the entire pipeline block
