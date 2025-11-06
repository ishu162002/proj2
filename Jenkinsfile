pipeline {
    agent any
    
    environment {
        GOOGLE_APPLICATION_CREDENTIALS = "${WORKSPACE}/gcp-key.json"
        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"
        PROJECT_ID = 'arched-proton-477313-g2'   // <-- replace with your actual GCP project ID
        IMAGE_NAME = 'healthcare'
        IMAGE_TAG = 'v1.0'
        REGION = 'us-central1'               // <-- replace with your VM/cluster region
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
               sh 'docker build -t ishupurwar/healthcare:latest .'
        }
    }
}
         stages {
        stage('Authenticate to GCP') {
            steps {
                // Write the service account key from Jenkins credentials to a file
                withCredentials([file(credentialsId: 'GCP_SERVICE_ACCOUNT_KEY', variable: 'KEY_FILE')]) {
                    sh 'cp $KEY_FILE $GOOGLE_APPLICATION_CREDENTIALS'
                }

                // Authenticate to GCP
                sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                sh 'gcloud config set project YOUR_PROJECT_ID'
            }
        }

        stage('Do GCP stuff') {
            steps {
                sh 'gcloud compute instances list' // Example command
            }
        }
    }
}



        stage('Deploy to Kubernetes Cluster') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        echo "Getting GKE credentials..."
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud container clusters get-credentials $CLUSTER_NAME --region $REGION --project $PROJECT_ID
                        
                        echo "Deploying to Kubernetes..."
                        kubectl apply -f k8s/deployment.yml
                        kubectl apply -f k8s/service.yml
                    '''
                }
            }
        }

        stage('Post Deployment Verification') {
            steps {
                echo 'Verifying Kubernetes Deployment...'
                sh '''
                    kubectl get pods
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful on GCP Kubernetes Cluster!'
        }
        failure {
            echo '❌ Pipeline Failed. Check logs for errors.'
        }
    }
}

