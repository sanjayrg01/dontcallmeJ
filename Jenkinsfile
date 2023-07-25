pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }

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
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        // Ensure the DockerHub credentials ID matches 'dockerhub-credentials' in the withRegistry block above
                        // 'dockerhub-credentials' is the ID of the Jenkins credential holding DockerHub username and password
                        // Use 'docker.withRegistry' for DockerHub
                        docker.login(usernameVar: 'DOCKERHUB_USERNAME', passwordVar: 'DOCKERHUB_PASSWORD')
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
