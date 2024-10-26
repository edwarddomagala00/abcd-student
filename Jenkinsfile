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
                docker run --name juice-shop -d --rm -p \
                3000:3000 bkimminich/juice-shop
                sleep 5
                '''

                sh '''
                docker run name --zap --rm
                  -v /home/mkopras/circle/DevSecOps/abcd-student/.zap:/zap/wrk/:rw
                  -t ghcr.io/zaproxy/zaproxy:stable \
                  /bin/bash -c "zap.sh -cmd -addonupdate; /bin/bash zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha --addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                 '''
            }
            post {
                always {
                    sh '''
                    docker cp zap::/zap/zap wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                    docker cp zap::/zap/zap wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                    docker stop zap juice-shop
                    docker rm zap
                    '''
                }   
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
                    echo "archiving"
                    archiveArtifacts artifacts: '/results/**/*', fingerprint: true, allowEmptyArchive: true
                    echo "TODO, publish results"
                }
            }
        }
    }
}
