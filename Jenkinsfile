pipeline {
    agent any

    stages {
        stage('Parallel Builds') {
            parallel {
                stage('Build Feature-Login') {
                    when { branch 'feature-login' }
                    steps {
                        echo "Building and testing feature-login branch..."
                        sleep 5
                        echo "Login build completed ✅"
                    }
                }

                stage('Build Feature-Payment') {
                    when { branch 'feature-payment' }
                    steps {
                        echo "Building and testing feature-payment branch..."
                        sleep 5
                        echo "Payment build completed ✅"
                    }
                }
            }
        }
    }
}
