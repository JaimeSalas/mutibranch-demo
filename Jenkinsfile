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
  }
}
