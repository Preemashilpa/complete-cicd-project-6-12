pipeline {
    agent any

    environment {
        // Make the Docker tag dynamic, using the build ID or Git commit hash
        DOCKER_TAG = "${env.BUILD_ID}"
        IMAGE_NAME = "preema21/fullstack"
        AWS_REGION = "ap-south-1"
        CLUSTER_NAME = "microdegree-cluster"
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    tools {
        jdk 'java-17'
        maven 'maven'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Preemashilpa/complete-cicd-project-6-12.git'
            }
        }

        stage('Compile & Build') {
            steps {
                // Clean, compile, and package the application
                sh "mvn clean package"
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    // Build and tag Docker image with the dynamic DOCKER_TAG
                    sh "docker build -t ${IMAGE_NAME}:${DOCKER_TAG} ."
                }
            }
        }

        // Commented out the Docker Image Scan stage as requested
        // stage('Docker Image Scan') {
        //     steps {
        //         script {
        //             // sh "trivy image --format table -o trivy-image-report.html ${IMAGE_NAME}:${DOCKER_TAG}"
        //         }
        //     }
        // }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Log in to Docker Hub with the credentials
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to Docker Hub
                    sh "docker push ${IMAGE_NAME}:${DOCKER_TAG}"
                }
            }
        }

        stage('Update the Cluster') {
            steps {
                withAWS(credentials: 'AWS_CREDENTIALS') {
                    // Update the kubeconfig for the cluster dynamically
                    sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}"
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(credentialsId: 'kube', namespace: 'microdegree') {
                    // Deploy the application to Kubernetes
                    sh "kubectl get pods -n microdegree"
                    sh "kubectl apply -f deployment.yml -n microdegree"
                }
            }
        }

        stage('Verify the Deployment') {
            steps {
                withKubeConfig(credentialsId: 'kube', namespace: 'microdegree') {
                    // Verify the deployed pods and services
                    sh "kubectl get pods -n microdegree"
                    sh "kubectl get svc -n microdegree"
                }
            }
        }
    }
}
