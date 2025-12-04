pipeline {
    // กำหนด Agent ที่มี Docker ติดตั้งอยู่
    agent any 

    // ตัวแปรและ Secrets ที่ใช้ใน Pipeline
    environment {
        // **URL ของแอปพลิเคชัน Staging ที่ต้องการสแกน**
        // อย่าลืมเปลี่ยนค่านี้เป็น URL ที่เข้าถึงได้จริงของ API
        TARGET_URL = 'http://api-staging.mydomain.com:8080' 
        
        // ชื่อไฟล์ ZAP Plan
        ZAP_PLAN_FILE = 'zap_plan.yaml' 
    }

    stages {
        
        stage('1. Update ZAP Automation Plan & Cleanup') {
            steps {
                script {
                    echo "Updating ${env.ZAP_PLAN_FILE} with target: ${env.TARGET_URL}"
                    
                    // --------------------------------------------------------------------------
                    // [แก้ไข 1] ใช้ sed แทนที่ Placeholder ใน zap_plan.yaml
                    // แทนที่ 'TARGET_URL_PLACEHOLDER' ด้วยค่าจาก TARGET_URL ของ Jenkins
                    // --------------------------------------------------------------------------
                    sh "sed -i 's|TARGET_URL_PLACEHOLDER|${env.TARGET_URL}|g' ${env.ZAP_PLAN_FILE}"
                    
                    // --------------------------------------------------------------------------
                    // [แก้ไข 2] เพิ่มขั้นตอนทำความสะอาดไฟล์ Configuration ของ ZAP
                    // ป้องกันปัญหา XML Fatal Error ที่เกิดจาก config.xml เสียหาย
                    // --------------------------------------------------------------------------
                    sh "rm -f config.xml add-ons-state.xml || true"
                }
            }
        }
        
        // ------------------------------------------------------------------------
        // Security Scan Stage
        // ------------------------------------------------------------------------
        stage('2. Run OWASP ZAP Automation Scan (Docker One-Shot)') {
            steps {
                script {
                    echo "Starting ZAP Automation Scan on ${env.TARGET_URL}..."
                    
                    // รัน ZAP Automation Framework (YAML) โดยใช้ Docker CLI
                    // -v \${PWD}:/zap/wrk/:rw: Map โฟลเดอร์ปัจจุบันของ Jenkins Worker เข้าสู่ Container
                    sh """
                        docker run --rm -u 0 -v \${PWD}:/zap/wrk/:rw zaproxy/zap-stable \
                        zap.sh -cmd -autorun /zap/wrk/${env.ZAP_PLAN_FILE} -dir /zap/wrk
                    """
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