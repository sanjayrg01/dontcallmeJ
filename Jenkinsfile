pipeline {
  agent any

    environment {
        // Customize these environment variables according to your project
        DOCKER_REGISTRY = "sanjayrg01" // e.g., docker.io or your private registry URL
        DOCKER_IMAGE_NAME = "mysite"
        DOCKER_IMAGE_TAG = ":v" // Add the DOCKER_IMAGE_TAG variable with a default value
        GITHUB_REPO_URL = "https://github.com/sanjayrg01/dontcallmeJ.git"
        KUBECONFIG_FILE_PATH = "kubeconfig" // Path to the kubeconfig file in the repository
        MANIFEST_FILE = "mysite.yaml"
        DOCKERHUB_CREDENTIALS = credentials('dockerhublogin') // Jenkins credential for DockerHub (username and password)
    }

    stages {
        stage('Clone') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'main']], userRemoteConfigs: [[url: "${env.GITHUB_REPO_URL}"]]])
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def versionNumber = "${env.DOCKER_IMAGE_NAME}-${env.BUILD_NUMBER}"
                    def dockerTag = "${env.DOCKER_REGISTRY}/${versionNumber}"

                    def dockerImage = docker.build(dockerTag, '.')
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhublogin', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        def registryURL = 'https://index.docker.io/v1/'
                        def dockerCredentials = 'dockerhublogin'

                        // Set up the Docker login command with --password-stdin
                        def loginCmd = "docker login -u ${DOCKERHUB_USERNAME} --password-stdin ${registryURL}"

                        // Run the Docker login command securely using sh and echo
                        sh "echo ${DOCKERHUB_PASSWORD} | ${loginCmd}"
                    }
                }
            }
        }


        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("${env.DOCKER_REGISTRY}", "docker-registry-credentials") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Copy the kubeconfig file to the Jenkins workspace
                    sh "cp ${env.WORKSPACE}/${env.KUBECONFIG_FILE_PATH} ${env.WORKSPACE}/kubeconfig"

                    // Set KUBECONFIG environment variable to point to the copied kubeconfig file
                    withEnv(["KUBECONFIG=${env.WORKSPACE}/kubeconfig"]) {
                        sh "kubectl apply -f ${env.MANIFEST_FILE}"
                    }
                }
            }
        }
    }
}
