pipeline {
  agent {label 'ubuntu'}
  stages {
    stage("verify tooling") {
      steps {
        sh '''
          echo 'docker version'
          sudo docker version
          echo 'docker info'
          sudo docker info
          echo 'compose version'
          sudo docker compose version || docker-compose version
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
        sh 'sudo docker compose up -d --wait'
        sh 'sudo docker compose ps'
      }
    }
    stage('Run tests against the container') {
      steps {
        sh 'curl http://${AGENTIP}/index.html '
      }
    }
  }
  post {
    always {
      sh 'sudo docker compose down --remove-orphans -v'
      sh 'sudo docker compose ps'
    }
  }
}