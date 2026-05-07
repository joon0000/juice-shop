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
        PROJECT_NAME = 'jenkins-juice_shop'

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
            }
        }

        stage('Lint') {
            steps {
                echo '====== Running lint ======'
            }
        }

        stage('Build Application') {
            steps {
                echo '====== Building application ======'
            }
        }

        stage('Unit Tests') {
            steps {
                echo '====== Running unit tests ======'
            }
        }

        stage('SAST Scan') {
            steps {
                echo '====== Running SAST scans ======'

                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    checkmarxASTScanner(
                        additionalOptions: '--scan-types sast',
                        baseAuthUrl: '',
                        branchName: "${GIT_BRANCH}",
                        checkmarxInstallation: 'CxAST CLI',
                        credentialsId: '',
                        projectName: "${PROJECT_NAME}",
                        serverUrl: '',
                        tenantName: '',
                        useOwnAdditionalOptions: true
                    )
                }
            }
        }

        stage('SCA Scan') {
            steps {
                echo '====== Running SCA scans ======'

                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    checkmarxASTScanner(
                        additionalOptions: '--scan-types sca',
                        baseAuthUrl: '',
                        branchName: "${GIT_BRANCH}",
                        checkmarxInstallation: 'CxAST CLI',
                        credentialsId: '',
                        projectName: "${PROJECT_NAME}",
                        serverUrl: '',
                        tenantName: '',
                        useOwnAdditionalOptions: true
                    )
                }
            }
        }        

        stage('Build Docker Image') {
            steps {
                echo '====== Building Docker image ======'
            }
        }

        stage('Push Docker Image') {

            when {
                branch 'main'
            }

            steps {
                echo '====== Pushing Docker image ======'
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

