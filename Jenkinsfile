pipeline {
    agent any

    environment {
        IMAGE_NAME = 'durveshy27/springrestxapi'
        DOCKERCREDENTIALS = credentials('docker-token')
        MINIKUBE_IP = '13.234.232.127'
    }

    tools {
        maven 'maven-3.9.11'
    }

    parameters {
        string(
            name: 'DEPLOY_ENV',
            defaultValue: 'development',
            description: 'Select the target environment'
        )
    }

    stages {

        stage('Checkout') {
            when {
                expression { params.DEPLOY_ENV == 'development' }
            }
            steps {
                checkout scm
                sh '''
                    echo "Checkout completed"
                    echo "DEPLOY_ENV = $DEPLOY_ENV"
                    ls -l
                '''
            }
        }

        stage('Check Tools') {
            steps {
                sh '''
                    echo "PATH = $PATH"
                    which mvn
                    mvn --version
                    java -version
                '''
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    echo "===== Building Java Application ====="
                    mvn clean package -B -DskipTests
                    echo "===== Build Completed ====="
                '''
            }
        }

        stage('Test Application (JUnit)') {
            steps {
                sh '''
                    echo "===== Running JUnit Tests ====="
                    mvn test
                '''
            }
            post {
                always {
                    echo "Publishing JUnit Test Results"
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "===== Building Docker Image ====="
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Scan Docker Image (Trivy)') {
            steps {
                sh '''
                    echo "===== Trivy Scan Started ====="
                    trivy image ${IMAGE_NAME}:${BUILD_NUMBER}
                    echo "===== Trivy Scan Completed ====="
                '''
            }
        }

        stage('Docker Login & Push') {
            steps {
                sh '''
                    echo "===== Logging into Docker Hub ====="
                    docker login -u $DOCKERCREDENTIALS_USR -p $DOCKERCREDENTIALS_PSW
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy to AWS EC2 (Minikube)') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh '''
                        echo "Connecting to EC2"
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ubuntu@$MINIKUBE_IP "echo Connected"

                        echo "Copying deployment.yaml"
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no deployment.yaml ubuntu@$MINIKUBE_IP:/home/ubuntu/

                        echo "Deploying to Kubernetes"
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ubuntu@$MINIKUBE_IP "
                            kubectl delete -f /home/ubuntu/deployment.yaml --ignore-not-found=true &&
                            kubectl apply -f /home/ubuntu/deployment.yaml
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
