pipeline {
  agent {label 'ubuntu'}
  environment {
    AGENTIP = sh 'hostname -I | awk \'{print $1}\''
  }
  stages {
    stage("verify tooling") {
      steps {
        sh '''
          echo 'AGENTIP: ${env.AGENTIP}
          echo 'docker version'
          sudo docker version
          echo 'docker info'
          sudo docker info
          echo 'compose version'
          sudo docker compose version || sudo docker-compose version
          sudo curl --version
        '''
      }
    }
    stage('Prune Docker data') {
      steps {
        sh 'sudo docker system prune -a --volumes -f'
      }
    }
    stage('Start container') {
      steps {
        sh 'sudo docker compose up -d --wait || sudo docker-compose up -d --wait'
        sh 'sudo docker compose ps || sudo docker-compose ps'
      }
    }
    stage('Run tests against the container') {
      steps {
        sh 'curl http://${env.AGENTIP}:9889'
      }
    }
  }
  post {
    always {
      sh 'sudo docker compose down --remove-orphans -v || sudo docker-compose down --remove-orphans -v'
      sh 'sudo docker compose ps || sudo docker-compose ps'
    }
  }
}