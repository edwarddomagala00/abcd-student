pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/edwarddomagala00/abcd-student', branch: 'main'
                }
            }
        }
        stage('DAST') {
            steps {
                sh '''
                docker rm -f juice-shop zap
                docker run --rm --name juice-shop --network demo-network -d -p 3000:3000 bkimminich/juice-shop

                sleep 5
                '''

                sh '''
                docker run --name zap --network demo-network \
                  -v /home/mkopras/circle/DevSecOps/abcd-student/.zap:/zap/wrk/:rw \
                  -t ghcr.io/zaproxy/zaproxy:stable \
                  /bin/bash -c "zap.sh -cmd -addonupdate; /bin/bash zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha --addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                  mkdir -p /tmp/results
                  docker cp zap:/zap/wrk/reports/zap_html_report.html /tmp/results/zap_html_report.html
                  docker cp zap:/zap/wrk/reports/zap_xml_report.xml /tmp/results/zap_xml_report.xml
                 '''
                 archiveArtifacts artifacts: '/tmp/results/*', fingerprint: true, allowEmptyArchive: true

            }
       }       
       stage('osv-scanner') {
            steps {
                sh '''
                osv-scanner scan --lockfile package-lock.json --format json --output osv.json || true
                ls -la
                head osv.json
                '''
            }
       post {
            always {
                 defectDojoPublisher(artifact: 'osv.json', 
                    productName: 'Juice Shop', 
                    scanType: 'OSV Scan', 
                    engagementName: 'mikolaj.kopras@gmail.com')
                  defectDojoPublisher(artifact: '/tmp/results/zap_xml_report.xml', 
                    productName: 'Juice Shop', 
                    scanType: 'ZAP Scan', 
                    engagementName: 'mikolaj.kopras@gmail.com')
         }
        }
    }
}
