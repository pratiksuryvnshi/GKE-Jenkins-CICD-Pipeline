pipeline {
    agent any

    environment {
        PROJECT_ID = 'k8s-jenkins-440812'
        IMAGE_NAME = "gcr.io/${PROJECT_ID}/my-repository/hello-cloudbuild"
        COMMIT_SHA = "${env.GIT_COMMIT}"
        CLUSTER_NAME = 'hello-cloudbuild'
        REGION = 'us-central1'
    }

    stages {
        stage('Test') {
            steps {
                script {
                    docker.image('python:3.7-slim').inside {
                        sh 'pip install flask'
                        sh 'python test_app.py -v'
                    }
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${COMMIT_SHA}")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'gcp-json-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}'
                        sh "docker push ${IMAGE_NAME}:${COMMIT_SHA}"
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'gcp-jenkins-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}'
                        sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${REGION} --project ${PROJECT_ID}"

                        // Replace placeholder in kubernetes.yaml and deploy
                        sh "sed 's|gcr.io/${PROJECT_ID}/my-repository/hello-cloudbuild:COMMIT_SHA|${IMAGE_NAME}:${COMMIT_SHA}|g' kubernetes.yaml > kubernetes_deploy.yaml"
                        sh "kubectl apply -f kubernetes_deploy.yaml"
                    }
                }
            }
        }
    }
}
