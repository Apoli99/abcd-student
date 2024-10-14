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
                    git credentialsId: 'github-pat', url: 'https://github.com/Apoli99/abcd-student', branch: 'main'
                }
            }
        }
        stage('Start Juice Shop') {
            steps {
                script {
                    sh '''
                        docker run --name juice-shop -d --rm \
                            -p 3000:3000 \
                            bkimminich/juice-shop
                    '''
                    sh 'sleep 5'
                }
            }
        }
       
        stage('ZAP Passive Scan') {
            steps {
                script {
                    sh '''
                    docker run --rm --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v /home/dawid/abcd-student/.zap:/zap/wrk:rw \
                    -v /home/dawid/Reports:/zap/wrk/reports:rw \
                    ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate && \
                    zap.sh -cmd -addoninstall communityScripts && \
                    zap.sh -cmd -addoninstall pscanrulesAlpha && \
                    zap.sh -cmd -addoninstall pscanrulesBeta && \
                    zap.sh -cmd -autorun /zap/wrk/passive.yaml"
                '''
                }
            }
        }
    }

        post {
            always {
                script{
                sh 'docker stop juice-shop || true'
                sh 'pwd'

                defectDojoPublisher(
                    artifact: 'zap_xml_report.xml', 
                    productName: 'Juice Shop', 
                    scanType: 'ZAP Scan', 
                    engagementName: 'dawid.apolinarski@enp.pl'
                )

            }
        }
    }
}
