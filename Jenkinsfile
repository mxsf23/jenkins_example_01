pipeline {
  agent {label 'ubuntu'}
  environment {
    AGENTADDR = sh(script: "hostname -I | awk \'{print \$1}\'",returnStdout: true).trim()
  }
  stages {
    stage("verify packages") {
      steps {
        sh '''
          echo "AGENTADDR: ${AGENTADDR}"
          printenv
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
    stage('Start container') {
      steps {
        sh 'sudo docker compose up -d --wait || sudo docker-compose up -d'
      }
    }
    stage('QA') {
      steps {
        script { 
          def resp_code = sh(script: "curl -s -o /dev/null -I -w \'%{http_code}\' http://${AGENTADDR}:9889",returnStdout: true).trim()
          echo "$resp_code"
          if ("$resp_code" == "200") { 
            echo "RESP CODE: $resp_code"
          } else {
            echo "Curl FAILED!!!"
            exit 1
          }   
        }
        script {
          def md5_curr = sh(script: "curl -s http://${AGENTADDR}:9889 | md5sum | cut -d ' ' -f 1",returnStdout: true).trim()
          echo "MD5 Current: ${md5_curr}"
          def md5_new = sh(script: "md5sum $WORKSPACE/files/index.html | cut -d ' ' -f 1",returnStdout: true).trim()
          echo "MD5 New: ${md5_new}"
          if ("$md5_curr" == "$md5_new") { 
              echo "MD5 are equal: $md5_curr"
          } else {
              echo "OOPS! MD5 Differs!"
          }
        }
      }
    }
  }
  post {
    always {
      sh 'sudo docker compose down --remove-orphans -v || sudo docker-compose down --remove-orphans -v'
      sh 'sudo docker compose ps || sudo docker-compose ps'
    }
    failure {
      mail body: "<b>Project Details:</b><br>Project: ${env.JOB_NAME} <br> Build Number: ${env.BUILD_NUMBER} <br> Build URL: ${env.BUILD_URL}", charset: 'UTF-8', from: 'jenkins@jen.local', mimeType: 'text/html', replyTo: '', subject: "ERROR CI: Project name -> ${env.JOB_NAME}", to: "${EMAIL_RECP}";   
    } 
  }
}