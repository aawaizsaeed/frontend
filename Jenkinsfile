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
                    sh "docker tag ${DOCKER_FE_IMAGE}:${imageTag} ${REPO-FE}:${imageTag}"
                }
            }
        }
        stage('Tagging') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_REGISTRY_CREDS', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        def imageTag = "latest-${env.BUILD_NUMBER}"
                        echo "Logging in to Docker registry"
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                        echo "Pushing Docker image with tag: ${imageTag}"
                        sh "docker push ${REPO_FE}:${imageTag}"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    def imageTag = "latest-${env.BUILD_NUMBER}"
                    echo "Running tests on Docker image with tag: ${imageTag}"
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"
                    sh "docker run -d --name ${CONTAINER_NAME} -p 80:80 ${REPO_1}:${imageTag}"
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
