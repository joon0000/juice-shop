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

        stage('Prepare OCI Image') {
            steps {
                echo '====== Pulling public image with Skopeo ======'

                sh '''
                    set -e

                    rm -rf oci-images
                    mkdir -p oci-images
                    
                    skopeo copy --override-os linux --override-arch amd64 \
                      docker://docker.io/bkimminich/juice-shop:latest \
                      oci:oci-images/juice-shop:latest

                    cat oci-images/juice-shop/index.json | python3 -m json.tool
                '''
            }
        }

        stage('Container Scan') {
            steps {
                echo '====== Running Container scans ======'

                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    checkmarxASTScanner(
                        additionalOptions: '--scan-types container-security --container-images "oci-dir:/var/lib/jenkins/workspace/container/oci-images/juice-shop" --containers-local-resolution',
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

        stage('Push Docker Image') {

            when {
                branch 'master'
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

