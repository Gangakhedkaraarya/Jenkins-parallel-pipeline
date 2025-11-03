// THIS IS THE CORRECT, GENERIC JENKINSFILE
// IT MUST BE PLACED IN *BOTH* BRANCHES (feature-login AND feature-payment)

pipeline {
    agent any

    stages {
        stage('1. Build') {
            steps {
                // This 'env.BRANCH_NAME' variable is provided by Jenkins
                echo "BUILDING: App 'payment-gateway' for branch ${env.BRANCH_NAME}"
                // Simulate a build
                sh 'sleep 5' 
            }
        }
        
        // This is the Type 2 Parallelism (Parallel Stages within one build)
        stage('2. Test (in Parallel)') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo "Running Unit Tests for ${env.BRANCH_NAME}..."
                        sh 'sleep 10' 
                        echo "Unit Tests Complete."
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo "Running Integration Tests for ${env.BRANCH_NAME}..."
                        sh 'sleep 12'
                        echo "Integration Tests Complete."
                    }
                }
            }
        }
        
        stage('3. Deploy') {
            steps {
                echo "DEPLOYING: Branch ${env.BRANCH_NAME} to staging."
                sh 'sleep 3'
            }
        }
    }
    
    post {
        always {
            echo "=========================================="
            echo "Pipeline Summary"
            echo "Branch: ${env.BRANCH_NAME}"
            echo "Build Number: ${env.BUILD_NUMBER}"
            echo "=========================================="
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Please check logs."
        }
    }
}
