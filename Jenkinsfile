
pipeline {
    agent { label 'Subha' }

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Getting source code from GitHub'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Installing Python dependencies'
                sh '''
       x             python3 --version
                    python3 -m venv .venv
                    .venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running automated tests'
                sh '''
                    .venv/bin/python -m unittest test_app -v
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Flask application on Azure Agent'

                sh '''
		    set -eu

		    echo "Agent hostname: $(hostname)"
                    echo "Agent user: $(whoami)"
                    echo "Workspace: $(pwd)"

                    echo "Restarting Flask systemd service..."
		    sudo systemctl restart flask-cicd

		    echo "Checking service status..."
		    sudo systemctl is-active --quiet flask-cicd

		    echo "Checking Flask health endpoint..."
		    .venv/bin/python -c \
			 "from urllib.request import urlopen; print(urlopen('http://127.0.0.1:5000/health', timeout=10).read().decode())"
		    echo "Deployment health check passed!" '''
            }
        }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
