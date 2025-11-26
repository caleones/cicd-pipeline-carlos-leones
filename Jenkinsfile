pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x scripts/build.sh'
                sh './scripts/build.sh'
            }
        }

        stage('Test') {
            steps {
                sh 'chmod +x scripts/test.sh'
                sh './scripts/test.sh'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName
                    if (env.BRANCH_NAME == 'main') {
                        imageName = 'nodemain'
                    } else if (env.BRANCH_NAME == 'dev') {
                        imageName = 'nodedev'
                    } else {
                        error "Rama no soportada: ${env.BRANCH_NAME}"
                    }

                    def tag = 'v1.0'
                    env.IMAGE_FULL = "${imageName}:${tag}"

                    sh "docker build -t ${env.IMAGE_FULL} ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerName
                    def hostPort

                    if (env.BRANCH_NAME == 'main') {
                        containerName = 'nodemain'
                        hostPort = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        containerName = 'nodedev'
                        hostPort = '3001'
                    } else {
                        error "Rama no soportada: ${env.BRANCH_NAME}"
                    }

                    sh """
                        CID=\$(docker ps -aq --filter "name=${containerName}")
                        if [ ! -z "\$CID" ]; then
                            docker rm -f \$CID || true
                        fi

                        docker run -d --name ${containerName} -p ${hostPort}:3000 ${env.IMAGE_FULL}
                    """
                }
            }
        }
    }
}
