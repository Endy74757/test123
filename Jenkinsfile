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
                    // เปลี่ยน ${TARGET_URL} ใน YAML แทนที่ 'target: ${TARGET_URL}'
                    sh "sed -i 's|target: \\$TARGET_URL|target: ${env.TARGET_URL}|g' ${env.ZAP_PLAN_FILE}"
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
                    // --rm: ลบ container หลังจากรันเสร็จ
                    // -v \${PWD}:/zap/wrk/:rw: Map โฟลเดอร์ปัจจุบันของ Jenkins Worker เข้าสู่ Container
                    sh """
                        docker run --rm -u 0 -v \${PWD}:/zap/wrk/:rw zaproxy/zap-stable \
                        zap.sh -cmd -autorun /zap/wrk/${env.ZAP_PLAN_FILE} -mkdir /zap/wrk
                    """
                    
                    // หมายเหตุ: ถ้า ZAP พบช่องโหว่ตามเงื่อนไข 'fail: true' ใน zap_plan.yaml
                    // คำสั่ง docker run จะ Exit ด้วย Non-zero Code ทำให้ Pipeline ล้มเหลวที่นี่
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