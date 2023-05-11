pipeline {
  agent any
  environment {
    imageName = 'jaimesalas/api-math:latest'
  }
  stages {
    stage('Install dependencies') {
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true
        }
      }
      steps {
        sh 'npm ci'
      }
    }
    stage('Tests') {
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true
        }
      }
      steps {
        sh 'npm test'
      }
    }
    stage('Build image & push it to DockerHub') {
      when {
        branch 'develop'
      }
      steps {
        script {
          def dockerImage = docker.build(imageName)
          withDockerRegistry([credentialsId: 'docker-hub', url: '']) {
            dockerImage.push()
            sh 'docker rmi $imageName'
          }
        }
      }
    }
    stage('Deploy to server') {
      when {
        branch 'develop'
      }
      environment {
        containerName = 'my-api-app'
        ec2Instance = 'ec2-15-188-130-249.eu-west-3.compute.amazonaws.com'
        appPort = 80
      }
      steps {
        withCredentials(
          bindings: [sshUserPrivateKey(
            credentialsId: 'ec2-ssh-credentials',
            keyFileVariable: 'identityFile',
            passphraseVariable: 'passphrase',
            usernameVariable: 'user'
          )
        ]) {
          script {
            sh '''
              ssh -o StrictHostKeyChecking=no -i $identityFile $user@$ec2Instance \
              APP_PORT=$appPort CONTAINER_NAME=$containerName IMAGE_NAME=$imageName bash < ./scripts/deploy.sh
            '''
          }
        }
      }
    }
  }
}
