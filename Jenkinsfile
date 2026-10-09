
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
                    if [ -f app.pid ]; then
                        kill "$(cat app.pid)" 2>/dev/null || true
                        rm -f app.pid
                    fi

                    nohup .venv/bin/python app.py \
                        > app.log 2>&1 < /dev/null &

                    echo $! > app.pid
                    sleep 3

                    .venv/bin/python -c \
                        "from urllib.request import urlopen; print(urlopen('http://127.0.0.1:5000/health').read().decode())"
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
