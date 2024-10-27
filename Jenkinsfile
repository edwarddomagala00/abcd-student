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
                docker rm -f juice-shop
                docker run --rm --name juice-shop --network demo-network -d -p 3000:3000 bkimminich/juice-shop

                sleep 5
                '''

                sh '''
                docker run --name zap --rm --network demo-network \
                  -v /home/mkopras/circle/DevSecOps/abcd-student/.zap:/zap/wrk/:rw \
                  -t ghcr.io/zaproxy/zaproxy:stable \
                  /bin/bash -c "zap.sh -cmd -addonupdate; /bin/bash zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha --addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                 '''
            }
            post {
                always {
                    sh '''
                    ls -lq 
                    pwd
                    ls -la /
                    ls /zap
                    docker ps                     
                    '''
                    archiveArtifacts artifacts: '/zap/wrk/reports/*', fingerprint: true, allowEmptyArchive: true
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
        }
    }
}
