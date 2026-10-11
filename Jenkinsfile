
pipeline {
    agent { label 'Subha' }

    options {
        timestamps()
        skipDefaultCheckout(true)
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
                    set -eu

                    echo "Python version:"
                    python3 --version

                    echo "Creating virtual environment..."
                    python3 -m venv .venv

                    echo "Upgrading pip..."
                    .venv/bin/python -m pip install --upgrade pip

                    echo "Installing requirements..."
                    .venv/bin/python -m pip install -r requirements.txt

                    echo "Build completed successfully!"
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running automated tests'

                sh '''
                    set -eu

                    .venv/bin/python -m unittest discover -v

                    echo "All tests passed!"
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Flask application on Azure VM'

                sh '''
                    set -eu

                    echo "Agent hostname: $(hostname)"
                    echo "Agent user: $(whoami)"
                    echo "Workspace: $(pwd)"

                    echo "Restarting Flask service..."
                    sudo systemctl restart flask-cicd

                    echo "Checking systemd service status..."
                    sudo systemctl is-active --quiet flask-cicd

                    echo "Checking Flask health endpoint..."

                    HEALTH_OK=0

                    for attempt in $(seq 1 10); do
                        if .venv/bin/python -c \
                            "from urllib.request import urlopen; print(urlopen('http://127.0.0.1:5000/health', timeout=3).read().decode())" \
                            2>/dev/null
                        then
                            HEALTH_OK=1
                            echo "Health check passed on attempt $attempt."
                            break
                        fi

                        echo "Attempt $attempt failed. Waiting 2 seconds..."
                        sleep 2
                    done

                    if [ "$HEALTH_OK" -ne 1 ]; then
                        echo "Deployment health check failed!"

                        sudo systemctl status flask-cicd --no-pager -l || true
                        sudo journalctl -u flask-cicd -n 30 --no-pager || true

                        exit 1
                    fi

                    echo "Deployment completed successfully!"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}