pipeline {
    agent any

    environment {
        APP_NAME = 'jenkins-github-demo'
        WEB_DIR  = '/var/tmp/cicd-webpage'
        WEB_PORT = '8081'
    }

    stages {
        stage('Checkout & Inspect') {
            steps {
                echo "Building ${APP_NAME} - Build #${BUILD_NUMBER}"
                sh '''
                    pwd
                    ls -la
                    git log -n 1 --oneline || true
                '''
            }
        }

        stage('Create & Test App') {
            steps {
                sh '''
                    mkdir -p src tests
                    cat << 'EOF' > src/app.sh
#!/bin/bash
echo "Hello from ${APP_NAME}! Build #${BUILD_NUMBER}"
EOF
                    chmod +x src/app.sh

                    ./src/app.sh > tests/output.txt
                    grep -q "Hello from ${APP_NAME}!" tests/output.txt
                    echo "Test passed: $(cat tests/output.txt)"
                '''
            }
        }

        stage('Package Artifact') {
            steps {
                sh '''
                    mkdir -p release
                    tar -czf release/${APP_NAME}-build-${BUILD_NUMBER}.tar.gz src/ tests/output.txt Jenkinsfile
                    ls -lh release/${APP_NAME}-build-${BUILD_NUMBER}.tar.gz
                '''
            }
        }

        stage('Deploy Webpage') {
            steps {
                sh '''
                    mkdir -p ${WEB_DIR}
                    cat << EOF > ${WEB_DIR}/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>${APP_NAME} - Build #${BUILD_NUMBER}</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 40px; background: #f4f6f9; color: #222; }
    .card { background: #fff; padding: 24px; border-radius: 8px; max-width: 600px; box-shadow: 0 2px 8px rgba(0,0,0,0.08); }
    .badge { display: inline-block; background: #16a34a; color: #fff; padding: 4px 10px; border-radius: 4px; font-size: 14px; }
  </style>
</head>
<body>
  <div class="card">
    <span class="badge">LIVE CI/CD</span>
    <h1>${APP_NAME}</h1>
    <p><strong>Latest Build:</strong> #${BUILD_NUMBER}</p>
    <p><strong>Test Output:</strong> Hello from ${APP_NAME}! Build #${BUILD_NUMBER}</p>
    <p>Deployed automatically via GitHub Webhook + Jenkins.</p>
  </div>
</body>
</html>
EOF

                    pkill -f "http.server ${WEB_PORT}" || true
                    nohup python3 -m http.server ${WEB_PORT} --directory ${WEB_DIR} > ${WEB_DIR}/server.log 2>&1 &
                    sleep 2
                    curl -s http://127.0.0.1:${WEB_PORT}/ | head -n 5
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: "release/*.tar.gz", fingerprint: true
            echo "SUCCESS: Build #${BUILD_NUMBER} completed, artifact archived, and webpage updated!"
        }
        failure {
            echo "FAILURE: Build #${BUILD_NUMBER} failed!"
        }
        cleanup {
            sh 'rm -rf src tests release'
        }
    }
}
