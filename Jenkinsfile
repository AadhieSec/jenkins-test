
pipeline {
    agent any

    environment {
        DOCKER_BIN  = '/Users/apple/.docker/bin/docker'
        DOCKER_HOST = 'unix:///Users/apple/.docker/run/docker.sock'
        IMAGE_NAME  = 'jenkins-lab'
        CONTAINER   = 'jenkins-lab'
        APP_PORT    = '8081'
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(
            numToKeepStr: '10'
        ))
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code checked out from GitHub'
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    $DOCKER_BIN build \
                        -t $IMAGE_NAME:$BUILD_NUMBER \
                        -t $IMAGE_NAME:latest \
                        .
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Testing application...'

                sh '''
                    test -f src/index.html
                    test -f Dockerfile

                    $DOCKER_BIN run --rm \
                        --entrypoint sh \
                        $IMAGE_NAME:$BUILD_NUMBER \
                        -c "test -f /usr/local/apache2/htdocs/index.html"

                    echo "Application validation passed!"
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Apache container...'

                sh '''
                    # Remove previous deployment
                    $DOCKER_BIN rm -f $CONTAINER || true

                    # Start new container
                    $DOCKER_BIN run -d \
                        --name $CONTAINER \
                        --restart unless-stopped \
                        -p $APP_PORT:80 \
                        $IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }

        stage('Verify') {
            steps {
                echo 'Verifying deployment...'

                sh '''
                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:$APP_PORT \
                            -o /tmp/jenkins-deploy-test.html; then
                            break
                        fi

                        echo "Waiting for Apache..."
                        sleep 2
                    done

                    test -s /tmp/jenkins-deploy-test.html

                    grep -q "Hello from Jenkins" \
                        /tmp/jenkins-deploy-test.html

                    echo "Deployment verification successful!"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
            echo 'Application: http://localhost:8081'
        }

        failure {
            echo 'CI/CD pipeline failed!'
            echo 'Check Console Output for errors.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
