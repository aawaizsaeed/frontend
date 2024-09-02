pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    def branchName = params.BRANCH
                    echo "Checking out branch: ${branchName}"

                    checkout([$class: 'GitSCM',
                        branches: [[name: "*/${branchName}"]],
                        userRemoteConfigs: [[url: "${MY_CODE-1}"]]
                    ])
                }
            }
        }
        stage('Building') {
            steps {
                script {
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    echo "Building Docker image with tag: ${imageTag}"
                    sh "docker build -t ${DOCKER_FE_IMAGE}:${imageTag} ."
                    sh "docker tag ${REPO-1}:${imageTag} ${REPO-1}:${imageTag}"
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    echo "Running tests on Docker image with tag: ${imageTag}"
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"
                    sh "docker run -d --name ${CONTAINER_NAME} -p 8000:5000 ${REPO-1}:${imageTag}"
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_REGISTRY_CREDS', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        def imageTag = "latest-${env.BUILD_NUMBER}"
                        echo "Logging in to Docker registry"
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                        echo "Pushing Docker image with tag: ${imageTag}"
                        sh "docker push ${REPO-1}:${imageTag}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Logging out from Docker registry"
            sh 'docker logout'
        }
    }
}
