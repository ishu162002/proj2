pipeline {
    agent any

    environment {
        GOOGLE_APPLICATION_CREDENTIALS = "${WORKSPACE}/gcp-key.json"
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
                withCredentials([file(credentialsId: 'GCP_SERVICE_ACCOUNT_KEY', variable: 'KEY_FILE')]) {
                    sh 'cp $KEY_FILE $GOOGLE_APPLICATION_CREDENTIALS'
                }
                sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                sh "gcloud config set project ${PROJECT_ID}"
            }
        }

        stage('Ensure GKE Cluster Exists') {
            steps {
                echo "Checking if GKE cluster ${CLUSTER_NAME} exists..."
                script {
                    def clusterExists = sh(
                        script: "gcloud container clusters list --region ${REGION} --project ${PROJECT_ID} --filter='name=${CLUSTER_NAME}' --format='value(name)'",
                        returnStdout: true
                    ).trim()

                    if (clusterExists == "") {
                        echo "Cluster not found. Creating GKE cluster..."
                        sh """
                            gcloud container clusters create ${CLUSTER_NAME} \
                                --region ${REGION} \
                                --num-nodes 3 \
                                --project ${PROJECT_ID}
                        """
                    } else {
                        echo "Cluster ${CLUSTER_NAME} already exists."
                    }
                }
            }
        }

        stage('Get GKE Credentials') {
            steps {
                sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${REGION} --project ${PROJECT_ID}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                sh '''
                    kubectl apply -f k8s/deployment.yml
                    kubectl apply -f k8s/service.yml
                '''
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

