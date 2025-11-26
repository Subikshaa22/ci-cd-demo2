pipeline {
    agent any

    environment {
        IMAGE = "subikshaa22/ci-cd-demo2:jenkins"
        VENV = ".venv"
        PYTHON = "python"   // Windows uses python.exe in PATH
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Subikshaa22/ci-cd-demo2',
                        credentialsId: 'github-creds'
                    ]]
                ])
            }
        }

        stage('Create Virtual Environment') {
            steps {
                bat """
                    ${PYTHON} -m venv ${VENV}
                """
            }
        }

        stage('Install Dependencies') {
            steps {
                bat """
                    ${VENV}\\Scripts\\pip install --upgrade pip
                    ${VENV}\\Scripts\\pip install -r requirements.txt
                """
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                    ${VENV}\\Scripts\\pytest -v
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                    docker build -t ${IMAGE} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'USER',
                                                  passwordVariable: 'PASS')]) {
                    bat """
                        echo ${PASS} | docker login -u ${USER} --password-stdin
                        docker push ${IMAGE}
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                bat """
                    docker pull ${IMAGE}
                    docker stop ci-cd-demo || exit 0
                    docker rm ci-cd-demo || exit 0
                    docker run -d -p 5000:5000 --name ci-cd-demo ${IMAGE}
                """
            }
        }
    }
}
