pipeline {
  agent any
  environment {
    REMOTE_USER    = 'ec2-user'
    REMOTE_HOST    = '54.236.246.122'
    DOCKER_IMAGE   = 'nginx-html-v1'
    CONTAINER_NAME = 'html-container'
    REMOTE_DIR     = "/home/${env.REMOTE_USER}/html-project"
  }

  triggers { githubPush() }

  stages {
    stage('Clone') {
      steps {
        git branch: 'master', url: 'https://github.com/mahadevprsd7/test-server.git'
      }
    }

    stage('Validate HTML') {
      steps {
        sh 'npm install -g html-validate'
        sh 'html-validate index.html'
      }
    }

    stage('Build Docker') {
      steps {
        sh "docker build -t ${DOCKER_IMAGE} ."
      }
    }

    stage('Deploy via SSH') {
      steps {
        sshagent(['ec2-ssh']) {
          sh """
            scp -o StrictHostKeyChecking=no index.html Dockerfile ${REMOTE_USER}@${REMOTE_HOST}:/tmp/
            ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
              rm -rf ${REMOTE_DIR}
              mkdir -p ${REMOTE_DIR}
              mv /tmp/index.html /tmp/Dockerfile ${REMOTE_DIR}/
              cd ${REMOTE_DIR}
              docker rm -f ${CONTAINER_NAME} || true
              docker build -t ${DOCKER_IMAGE} .
              docker run -d --name ${CONTAINER_NAME} -p 8080:80 ${DOCKER_IMAGE}
            EOF
          """
        }
      }
    }
  }

  post {
    success { echo "Deployment at http://${env.REMOTE_HOST}:8080" }
    failure { echo "Build or deployment failed." }
  }
}
