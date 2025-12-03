// Jenkinsfile
pipeline {
    // กำหนด Agent ที่มี Docker ติดตั้งอยู่
    agent any 

    // ตัวแปรและ Secrets ที่ใช้ใน Pipeline
    environment {
        // URL ของแอปพลิเคชัน Staging ที่ต้องการสแกน
        TARGET_URL = 'http://www.google.com' 
        
        // ชื่อไฟล์ ZAP Plan ที่กำหนด Logic การสแกน
        ZAP_PLAN_FILE = 'zap_plan.yaml' 
    }

    stages {
        
        stage('1. Update ZAP Automation Plan') {
            steps {
                script {
                    echo "Updating ${env.ZAP_PLAN_FILE} with target: ${env.TARGET_URL}"
                    // ใช้คำสั่ง sed เพื่อแทนที่ Target URL ในไฟล์ YAML
                    // *สมมติว่าใน zap_plan.yaml มีการกำหนด target: "" เพื่อรอการแทนที่
                    sh "sed -i 's|target: \"\"|target: \"${env.TARGET_URL}\"|g' ${env.ZAP_PLAN_FILE}"
                }
            }
        }
        
        // ------------------------------------------------------------------------
        // Security Scan Stage
        // ------------------------------------------------------------------------
        stage('2. Run OWASP ZAP Automation Scan (Docker One-Shot)') {
            steps {
                docker.image('owasp/zap2docker-stable').inside {
                    sh 'echo "Running ZAP inside the ZAP container..."'
                    // คำสั่งนี้จะถูกรันภายใน container ของ ZAP
                    // *หมายเหตุ: ต้องปรับการ map volumes ให้เหมาะสมกับไวยากรณ์ Groovy/inside*
                    // ในกรณีนี้ ไฟล์ zap_plan.yaml ควรถูกคัดลอกหรือสร้างขึ้นใน Agent 
                    // แล้วจึงรันภายใน Container
                    sh "zap.sh -autorun /var/jenkins_home/workspace/test_owasp_zap/zap_plan.yaml"
                }
            }
        }
        
        // ------------------------------------------------------------------------
        // Reporting Stage
        // ------------------------------------------------------------------------
        stage('3. Publish Report') {
            steps {
                // เก็บรายงาน (zap_report.html) ที่ถูกสร้างโดย ZAP ใน Stage ที่ 2
                archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
            }
        }
    }
}