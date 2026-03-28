pipeline {
    agent {
        label 'linux'
    }

    triggers {
        githubPush()
    }

    environment {
        // ⚠️ เปลี่ยน 'your_docker_id' เป็น Username ในเว็บ Docker Hub ของคุณนะครับ
        APP_NAME    = 'siridet4826/my-nginx-web'
        IMAGE_TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${APP_NAME}:${IMAGE_TAG} -t ${APP_NAME}:latest ."
                }
            }
        }

        // 🚀 เพิ่ม Stage นี้เข้ามาใหม่ เพื่อส่ง Image ขึ้น Docker Hub
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "docker push ${APP_NAME}:${IMAGE_TAG}"
                    sh "docker push ${APP_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // ❌ ลบ kind load ทิ้งไปเลย รันแค่คำสั่งพวกนี้พอครับ
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/service.yaml"
                    sh "kubectl apply -f k8s/ingress.yaml"
                    sh "kubectl set image deployment/nginx-deployment nginx-container=${APP_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh "kubectl rollout status deployment/nginx-deployment --timeout=120s"
                    sh "kubectl get pods -l app=my-nginx"
                    sh "kubectl get svc nginx-service"
                    sh "kubectl get ingress nginx-ingress"
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful! Access at http://my-nginx.local"
        }
        failure {
            echo "Deployment failed! Check logs for details."
        }
    }
}