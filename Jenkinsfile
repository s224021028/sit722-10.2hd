pipeline {
    agent any
    environment {
        USE_GKE_GCLOUD_AUTH_PLUGIN = "True"
        GCP_PROJECT = "sit722-10-2-hd"
        REGION = "us-central1-c"
        CLUSTER_NAME = "cluster-1"
        REPO_NAME = "s224021028gcr"
    }
    stages {
        stage("Login to Google Cloud") {
            steps {
                withCredentials([file(credentialsId: "ea148254-95f9-4aec-b8b4-2fa0dc449d50", variable: "GOOGLE_APPLICATION_CREDENTIALS")]) {
                    bat "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                    bat "gcloud config set project ${GCP_PROJECT}"
                }
            }
        }
        stage("Build Docker Images") {
            steps {
                dir("./gateway") {
                    bat "docker build -t gateway:latest --file=Dockerfile-prod ."
                }
                dir("./history") {
                    bat "docker build -t history:latest --file=Dockerfile-prod ."
                }
                dir("./metadata") {
                    bat "docker build -t metadata:latest --file=Dockerfile-prod ."
                }
                dir("./mock-storage") {
                    bat "docker build -t mock-storage:latest --file=Dockerfile-prod ."
                }
                dir("./video-streaming") {
                    bat "docker build -t video-streaming:latest --file=Dockerfile-prod ."
                }
                dir("./video-upload") {
                    bat "docker build -t video-upload:latest --file=Dockerfile-prod ."
                }
            }
        }
        stage("Tag Built Docker Images") {
            steps {
                bat "gcloud auth configure-docker ${REGION}-docker.pkg.dev"
                bat "docker tag gateway:latest ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/gateway:latest"
                bat "docker tag history:latest ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/history:latest"
                bat "docker tag metadata:latest ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/metadata:latest"
                bat "docker tag mock-storage:latest ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/mock-storage:latest"
                bat "docker tag video-streaming:latest ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/video-streaming:latest"
                bat "docker tag video-upload:latest ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/video-upload:latest"
            }
        }
        stage("Push images to Google Cloud Artifact Registry") {
            steps {
                bat "docker push ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/gateway:latest"
                bat "docker push ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/history:latest"
                bat "docker push ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/metadata:latest"
                bat "docker push ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/mock-storage:latest"
                bat "docker push ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/video-streaming:latest"
                bat "docker push ${REGION}-docker.pkg.dev/${GCP_PROJECT}/${REPO_NAME}/video-upload:latest"
            }
        }
        stage("Configure kubectl for GKE") {
            steps {
                bat "gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${REGION} --project ${GCP_PROJECT}"
            }
        }
        stage("Deploy app to GKE - FlixTube") {
            steps {
                bat "kubectl apply -f scripts/cd/gateway.yaml"
                bat "kubectl apply -f scripts/cd/history.yaml"
                bat "kubectl apply -f scripts/cd/metadata.yaml"
                bat "kubectl apply -f scripts/cd/mock-storage.yaml"
                bat "kubectl apply -f scripts/cd/video-streaming.yaml"
                bat "kubectl apply -f scripts/cd/video-upload.yaml"
            }
        }
        stage("Monitoring and Alerting using Grafana") {
            steps {
                script {
                    bat "helm repo add bitnami https://charts.bitnami.com/bitnami"
                    bat "helm upgrade --install stories-grafana bitnami/grafana --set service.type=LoadBalancer"
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
