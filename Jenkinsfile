pipeline {
    agent any

    environment {
        PYTHONPATH = "${WORKSPACE}"
        PAGER = "cat"
    }

    stages {
        stage('Setup Environment') {
            steps {
                dir('/Users/s.d./main/Personal/College/Practical-API-TESTING') {
                    echo 'Checking Python installation...'
                    sh 'python3 --version'
                    
                    echo 'Preparing virtual environment...'
                    sh '''
                        if [ ! -d "venv" ]; then
                            python3 -m venv venv
                        fi
                    '''
                    
                    echo 'Installing Selenium dependency...'
                    sh '''
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install selenium
                    '''
                }
            }
        }

        stage('Start Backend Server') {
            steps {
                dir('/Users/s.d./main/Personal/College/Practical-API-TESTING') {
                    echo 'Freeing up port 8000 if occupied...'
                    sh 'lsof -t -i:8000 | xargs kill -9 || true'

                    echo 'Starting main.py server in background...'
                    sh '''
                        . venv/bin/activate
                        JENKINS_NODE_COOKIE=dontKillMe python3 main.py > server.log 2>&1 &
                        echo $! > server.pid
                    '''
                    
                    echo 'Waiting for server to become responsive at http://127.0.0.1:8000...'
                    sh '''
                        . venv/bin/activate
                        python3 -c "
import urllib.request, time
for i in range(30):
    try:
        if urllib.request.urlopen('http://127.0.0.1:8000/', timeout=2).status == 200:
            print('Server is active!')
            break
    except Exception:
        time.sleep(1)
else:
    try:
        print('--- SERVER LOGS ---')
        with open('server.log', 'r') as f:
            print(f.read())
    except Exception:
        pass
    raise SystemExit('Server failed to start')
"
                    '''
                }
            }
        }

        stage('Run Selenium API Tests') {
            steps {
                dir('/Users/s.d./main/Personal/College/Practical-API-TESTING') {
                    sh '''
                        . venv/bin/activate
                        
                        echo "Executing Test 1: GET Users..."
                        python3 selenium_get_users.py
                        
                        echo "Executing Test 2: POST User (Signup)..."
                        python3 selenium_post_user.py
                        
                        echo "Executing Test 3: GET Messages..."
                        python3 selenium_get_messages.py
                        
                        echo "Executing Test 4: POST Message (Send)..."
                        python3 selenium_post_message.py
                        
                        echo "Executing Test 5: PUT Message (Edit)..."
                        python3 selenium_put_message.py
                        
                        echo "Executing Test 6: DELETE Message (Recall)..."
                        python3 selenium_delete_message.py
                    '''
                }
            }
        }
    }

    post {
        always {
            dir('/Users/s.d./main/Personal/College/Practical-API-TESTING') {
                echo 'Cleaning up server...'
                sh '''
                    if [ -f "server.pid" ]; then
                        PID=$(cat server.pid)
                        echo "Stopping main.py background process $PID"
                        kill $PID || true
                        rm server.pid
                    fi
                '''
                archiveArtifacts artifacts: 'server.log', allowEmptyArchive: true
            }
        }
    }
}
