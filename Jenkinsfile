pipeline {
    agent any

    environment {
        AWS_CREDENTIALS_ID = 'your-aws-credentials-id'
        REGION = 'us-west-2'  // Change to your AWS region
        EKS_CLUSTER_NAME = 'my-eks-cluster'
        DESTROY_RESOURCES = 'false'  // Set to 'true' to destroy resources
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Checkout the code from the repository
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    // Initialize Terraform with the AWS credentials
                    withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: "${env.REGION}")]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    // Generate the Terraform plan
                    withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: "${env.REGION}")]) {
                        sh 'terraform plan -out=tfplan'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            when {
                expression { env.DESTROY_RESOURCES == 'false' }  // Only apply if resources are not set to destroy
            }
            steps {
                script {
                    // Apply the Terraform plan to create the EKS cluster
                    withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: "${env.REGION}")]) {
                        sh 'terraform apply --auto-approve tfplan'
                    }
                }
            }
        }

        stage('Configure kubectl') {
            when {
                expression { env.DESTROY_RESOURCES == 'false' }
            }
            steps {
                script {
                    // Fetch the EKS kubeconfig after the cluster is created
                    withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: "${env.REGION}")]) {
                        sh """
                            aws eks --region ${env.REGION} update-kubeconfig --name ${env.EKS_CLUSTER_NAME}
                        """
                    }
                }
            }
        }

        stage('Terraform Destroy') {
            when {
                expression { env.DESTROY_RESOURCES == 'true' }  // Only destroy if the flag is set to 'true'
            }
            steps {
                script {
                    // Destroy resources using Terraform
                    echo "Destroying all resources..."
                    withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: "${env.REGION}")]) {
                        sh 'terraform destroy --auto-approve'
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Cleanup workspace after the pipeline
                echo 'Cleaning up workspace...'
                cleanWs()
            }
        }

        success {
            echo 'Pipeline succeeded!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
