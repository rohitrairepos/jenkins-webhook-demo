pipeline {
    agent {
        label 'control-built-in'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy Website') {
            steps {
                sh '''
                    echo "Deploying website..."
                    echo "whoami"

                    rm -rf /var/www/html/*
                    cp index.html /var/www/html/

                    echo "Website deployed successfully"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Checking website..."

                    curl -f http://localhost

                    echo ""
                    echo "Website is working!"
                '''
            }
        }
    }

    post {
        success {
            echo 'Website deployment successful!'
        }

        failure {
            echo 'Website deployment failed!'
        }
    }
}
