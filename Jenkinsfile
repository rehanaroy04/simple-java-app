pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rehanaroy04/java-app"
        KUBECONFIG = "C:\\Users\\Durable\\.kube\\config"
    }

    stages {
        // SET 1: Verify Connection to Kubernetes
        stage('Set 1: Verify K8s Connection') {
            steps {
                echo "Verifying Kubernetes cluster connection..."
                bat 'kubectl cluster-info'
            }
        }

        // SET 15 & 12: Build & Push Docker Image
        stage('Build and Push Docker Image') {
            steps {
                bat 'mvn clean package'
                bat "docker build -t %DOCKER_IMAGE%:latest ."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat """
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        // SET 2, 4, 5, 8, 9, 10: Deploy Everything
        stage('Deploy Manifests to K8s') {
            steps {
                echo "Applying ConfigMap, PVC, Deployment, and Service..."
                bat 'kubectl apply -f k8s/configmap.yaml'
                bat 'kubectl apply -f k8s/pvc.yaml'
                bat 'kubectl apply -f k8s/deployment.yaml'
                bat 'kubectl apply -f k8s/service.yaml'
            }
        }

        // SET 3: Create Standalone Pod
        stage('Set 3: Create Standalone Pod') {
            steps {
                bat 'kubectl apply -f k8s/pod.yaml'
            }
        }

        // SET 13: Verify Rolling Update
        stage('Set 13: Verify Rollout Status') {
            steps {
                echo "Waiting for rolling update to complete..."
                bat 'kubectl rollout status deployment/java-app'
            }
        }

        // SET 6: Scale Deployment
        stage('Set 6: Scale Deployment to 3') {
            steps {
                bat 'kubectl scale deployment/java-app --replicas=3'
            }
        }

        // SET 7: Update Deployment with New Image
        stage('Set 7: Update Image Tag') {
            steps {
                // We pretend we have a v2 image
                bat "kubectl set image deployment/java-app java-app=%DOCKER_IMAGE%:latest"
            }
        }
    }

    // SET 14: Troubleshoot Failures
    post {
        failure {
            echo "Build failed! Gathering Kubernetes troubleshooting info..."
            bat 'kubectl get pods'
            bat 'kubectl describe pods -l app=java-app'
            bat 'kubectl logs -l app=java-app --tail=50'
        }
        success {
            echo "Pipeline completed successfully! All sets executed."
        }
    }
}