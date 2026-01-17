@Library('Shared') _

pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = 'vignesh997/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'vignesh997/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanupWorkspace()
            }
        }

        stage('Clone Repository') {
            steps {
                checkoutRepo()
            }
        }

        stage('Build Docker Images') {
            parallel {

                stage('Build Main App Image') {
                    steps {
                        buildDockerImage(
                            imageName: DOCKER_IMAGE_NAME,
                            imageTag: DOCKER_IMAGE_TAG,
                            dockerfile: 'Dockerfile',
                            context: '.'
                        )
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        buildDockerImage(
                            imageName: DOCKER_MIGRATION_IMAGE_NAME,
                            imageTag: DOCKER_IMAGE_TAG,
                            dockerfile: 'scripts/Dockerfile.migration',
                            context: '.'
                        )
                    }
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                runUnitTests()
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                trivyScan()
            }
        }

        stage('Push Docker Images') {
            parallel {

                stage('Push Main App Image') {
                    steps {
                        pushDockerImage(
                            imageName: DOCKER_IMAGE_NAME,
                            imageTag: DOCKER_IMAGE_TAG,
                            credentials: 'docker-hub-credentials'
                        )
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        pushDockerImage(
                            imageName: DOCKER_MIGRATION_IMAGE_NAME,
                            imageTag: DOCKER_IMAGE_TAG,
                            credentials: 'docker-hub-credentials'
                        )
                    }
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                updateK8sManifests(
                    imageTag: DOCKER_IMAGE_TAG,
                    manifestsPath: 'kubernetes',
                    gitCredentials: 'github-credentials',
                    gitUserName: 'Jenkins CI',
                    gitUserEmail: 'vignesh.murugan1312gmail.com'
                )
            }
        }
    }
}
