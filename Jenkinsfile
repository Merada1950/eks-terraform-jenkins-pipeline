pipeline {
    agent any  // Use any available agent

    environment {
        // Define AWS credentials and environment variables
        // Example: AWS credentials ID for use with Terraform
        AWS_CREDENTIALS_ID = '339712843218'
        
        // Set this to 'true' to destroy the resources
        DESTROY_RESOURCES = 'false'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Checkout code from the repository
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    // Initialize Terraform
                    withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: 'us-west-1')]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    // Generate Terraform plan
                    withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: 'us-west-1')]) {
                        sh 'terraform plan -out=plan'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    // Apply the Terraform plan with auto-approval
                    withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: 'us-west-1')]) {
                        sh 'terraform destroy --auto-approve tfplan'
                    }
                }
            }
        }

    stage('Terraform Destroy') {
        steps {
            script {
               if (env.DESTROY_RESOURCES == 'true') {
                   withCredentials([aws(credentialsId: "${env.AWS_CREDENTIALS_ID}", region: 'us-west-1')]) {
                       sh 'terraform destroy --auto-approve'
                  }
               } else {
                   echo "skipping Terraform destroy as DESTROYED_RESOURCES is set to false."
                }
                }
            }
        }   
    }              
                   
    post {
        always {
            script {
                //cleanup workspace or other post-build actions
                echo 'cleaning up...'
                cleanWs()
              }
         }
                
        success {
            echo 'Pipeline succeeded|'
        }
    
    failure {
        echo 'Pipeline failed!'
    }
 }
}
          
