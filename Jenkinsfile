pipeline {
    agent any

    environment {
        DOCKER_HOST   = 'unix:///Users/apple/.docker/run/docker.sock'
        PATH = "/Users/apple/.docker/bin:/bin:/usr/bin:/usr/local/bin:/opt/homebrew/bin"
        DOCKER_CONFIG = '/Users/apple/.jenkins-docker'

        IMAGE_NAME = 'jenkins-lab'
        CONTAINER  = 'jenkins-lab'
        APP_PORT   = '8081'
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out successfully'
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building Docker image...'

                    def image = docker.build(
                        "${IMAGE_NAME}:${BUILD_NUMBER}"
                    )

                    echo "Image built: ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Checking application files..."

                    test -f src/index.html
                    test -f Dockerfile

                    echo "Testing Docker image..."

                    /Users/apple/.docker/bin/docker run --rm \
                        --entrypoint sh \
                        $IMAGE_NAME:$BUILD_NUMBER \
                        -c "test -f /usr/local/apache2/htdocs/index.html"

                    echo "Tests passed!"
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying application...'

                    sh '''
                        /Users/apple/.docker/bin/docker rm -f $CONTAINER || true
                    '''

                    def image = docker.image(
                        "${IMAGE_NAME}:${BUILD_NUMBER}"
                    )

                    image.run(
                        "--name ${CONTAINER} --restart unless-stopped -p ${APP_PORT}:80"
                    )
                }
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    echo "Verifying deployment..."

                    for i in $(seq 1 15); do
                        if curl -fsS http://localhost:$APP_PORT \
                            -o /tmp/jenkins-test.html; then
                            break
                        fi

                        echo "Waiting for Apache..."
                        sleep 2
                    done

                    test -s /tmp/jenkins-test.html
                    grep -q "Hello from Jenkins" /tmp/jenkins-test.html

                    echo "Deployment verified successfully!"
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
            echo 'CI/CD pipeline failed. Check Console Output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}