pipeline {
    agent any

    environment {
        PROJECT_ID = 'spheric-subject-482019-e5'
        REPO = 'my-docker-repo'                // GCP Artifact Registry repo
        IMAGE_NAME = 'myflaskapp'
        IMAGE_TAG = "${BUILD_NUMBER}"          // Versioning using Jenkins build number
        GCP_CREDENTIALS = 'gcp-sa'             // Jenkins credential ID for GCP service account JSON
        ZONE = 'us-central1-a'                 // GCP VM zone
        VM_NAME = 'my-vm-name'                 // GCP VM name
    }

    stages {

        stage('Checkout Code') {
            steps {
                // Use GitHub credential for authentication
                git branch: 'master', 
                    url: 'https://github.com/shekhar8595/spheric-subject-482019-e5.git',
                    credentialsId: 'jenkins-github'
            }
        }

        stage('Authenticate with GCP') {
            steps {
                withCredentials([file(credentialsId: "${GCP_CREDENTIALS}", variable: 'GCP_KEY')]) {
                    sh 'gcloud auth activate-service-account --key-file=$GCP_KEY'
                    sh 'gcloud config set project $PROJECT_ID'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t us-central1-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:$IMAGE_TAG ."
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push us-central1-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:$IMAGE_TAG"
            }
        }

        stage('Deploy to GCP VM') {
            steps {
                sh """
                gcloud compute ssh $VM_NAME --zone=$ZONE --command="
                    docker stop $IMAGE_NAME || true
                    docker rm $IMAGE_NAME || true
                    docker pull us-central1-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:$IMAGE_TAG
                    docker run -d -p 5000:5000 --name $IMAGE_NAME us-central1-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:$IMAGE_TAG
                "
                """
            }
        }
    }
}
