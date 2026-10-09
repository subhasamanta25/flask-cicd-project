
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
                    python3 --version
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

                    if [ -f app.pid ]; then
			OLD_PID=$(cat app.pid)
                        kill "$(cat app.pid)" 2>/dev/null || true
                        rm -f app.pid
                    fi

                    nohup .venv/bin/python app.py \
                        > app.log 2>&1 < /dev/null &

		    APP_PID=$!
                    echo "$APP_PID" > app.pid

                    sleep 3

		    if ! kill -0 "$APP_PID" 2>/dev/null; then
			 echo "Flask failed to start. Application log:"
			 cat app.log
			 exit 1
		    fi 

                    echo "Checking Flask health endpoint..."
                    .venv/bin/python -c \
                        "from urllib.request import urlopen; print(urlopen('http://127.0.0.1:5000/health').read().decode())"

                    echo "Application log:"
                    tail -n 20 app.log

		    echo "Deployment health cheak passed"
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
    }
}
