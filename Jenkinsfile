pipeline {
    agent any

    environment {
        PROJECT_ID = 'spheric-subject-482019-e5'
        REPO = 'my-docker-repo'
        IMAGE_NAME = 'myflaskapp'
        IMAGE_TAG = "${BUILD_NUMBER}"
        GCP_CREDENTIALS = 'gcp-sa'
        ZONE = 'europe-west1-b'
        VM_NAME = 'jenkins-devops-vm'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master', 
                    url: 'https://github.com/shekhar8595/docker-repo.git',
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

        stage('Authenticate Docker with GCP') {
            steps {
                withCredentials([file(credentialsId: "${GCP_CREDENTIALS}", variable: 'GCP_KEY')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud auth configure-docker us-central1-docker.pkg.dev --quiet
                    """
                }
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
                    sudo docker stop $IMAGE_NAME || true
                    sudo docker rm $IMAGE_NAME || true
                    sudo docker pull us-central1-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:$IMAGE_TAG
                    sudo docker run -d -p 5000:5000 --name $IMAGE_NAME us-central1-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:$IMAGE_TAG
                "
                """
            }
        }

        stage('Cleanup Environment') {
            steps {
                echo "Cleaning up Docker container, image, and VM..."
                sh """
                # Stop and remove container on VM
                gcloud compute ssh $VM_NAME --zone=$ZONE --command="
                    sudo docker stop $IMAGE_NAME || true
                    sudo docker rm $IMAGE_NAME || true
                "
                # Delete Docker image from Artifact Registry
                gcloud artifacts docker images delete us-central1-docker.pkg.dev/$PROJECT_ID/$REPO/$IMAGE_NAME:$IMAGE_TAG --delete-tags --quiet
                # Delete the VM
                gcloud compute instances delete $VM_NAME --zone=$ZONE --quiet
                """
            }
        }
    }
}
