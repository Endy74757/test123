def scmVars

pipeline {
  agent any
  stages{
    stage('checkout') {
      steps {
          script{
            scmVars = git branch: 'main', url: 'https://github.com/Endy74757/test123.git'
        }
      }
    }

    stage('Security Scan') {
      steps {
          script{
            docker.image('owasp/zap2docker-stable').inside {
                        sh 'echo "Starting ZAP Baseline Scan..."'
                        // ใช้คำสั่ง zap-baseline.py
                        sh 'zap-baseline.py -t https://google.com -r zap_report.html || true'
                        
                        // เก็บรายงานไว้
                        archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
                    }
          }
      }
    }   
  }
}

