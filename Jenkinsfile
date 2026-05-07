pipeline {
    agent any
    
    triggers {
        pollSCM('H/1 * * * *')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
        timestamps()
    }

    environment {
        NODE_ENV = 'test'

        // Docker Registry
        REGISTRY = 'docker.io'
        DOCKER_NAMESPACE = 'your-dockerhub-user'
        IMAGE_NAME = 'juice-shop'
        IMAGE_TAG = "${BUILD_NUMBER}"

        FULL_IMAGE = "${REGISTRY}/${DOCKER_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG}"
        LATEST_IMAGE = "${REGISTRY}/${DOCKER_NAMESPACE}/${IMAGE_NAME}:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                echo '====== Checking out source code ======'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '====== Installing dependencies ======'

                sh '''
                    npm ci

                    cd frontend
                    npm ci
                    cd ..
                '''
            }
        }

        stage('Lint') {
            steps {
                echo '====== Running lint ======'

                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh '''
                        npm run lint
                    '''
                }
            }
        }

        stage('Build Application') {
            steps {
                echo '====== Building application ======'

                sh '''
                    npm run build

                    cd frontend
                    npm run build
                    cd ..
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                echo '====== Running unit tests ======'

                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh '''
                        npm test
                    '''
                }

                junit allowEmptyResults: true, testResults: 'test-results/**/*.xml'
            }
        }

        stage('Security Scan') {
            steps {
                echo '====== Running security scans ======'

                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh '''
                        echi "checkmarx here"
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '====== Building Docker image ======'

                sh '''
                    docker build -t ${FULL_IMAGE} .
                    docker tag ${FULL_IMAGE} ${LATEST_IMAGE}
                '''
            }
        }

        stage('Push Docker Image') {

            when {
                branch 'main'
            }

            steps {
                echo '====== Pushing Docker image ======'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin ${REGISTRY}

                        docker push ${FULL_IMAGE}
                        docker push ${LATEST_IMAGE}

                        docker logout ${REGISTRY}
                    '''
                }
            }
        }

    }

    post {

        always {

            echo '====== Cleanup workspace ======'

            cleanWs()

            sh '''
                docker image prune -f || true
            '''
        }

        success {
            echo '====== Pipeline completed successfully ======'
        }

        unstable {
            echo '====== Pipeline completed with warnings ======'
        }

        failure {
            echo '====== Pipeline failed ======'
        }
    }
}

