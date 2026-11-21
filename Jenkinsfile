pipeline {
    agent any

    environment {
        APP_NAME = 'jenkins-github-demo'
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
    }

    post {
        success {
            archiveArtifacts artifacts: "release/*.tar.gz", fingerprint: true
            echo "SUCCESS: Build #${BUILD_NUMBER} completed and artifact archived!"
        }
        failure {
            echo "FAILURE: Build #${BUILD_NUMBER} failed!"
        }
        always {
            sh 'rm -rf src tests release'
        }
    }
}
