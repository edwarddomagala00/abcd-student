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
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
           }
        }
        stage('test') {
            steps {
                sh '''
                osv-scanner scan --json --lockfile package-lock.json > osv.json || true
                ls -la
                head osv.json
                '''
            }
        }
    }
}
