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

        stage('Prepare') {
            steps {
                sh 'mkdir -p results/' 
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
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v /home/dawid/abcd-student/.zap:/zap/wrk:rw \
                    ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate && \
                    zap.sh -cmd -addoninstall communityScripts && \
                    zap.sh -cmd -addoninstall pscanrulesAlpha && \
                    zap.sh -cmd -addoninstall pscanrulesBeta && \
                    zap.sh -cmd -autorun /zap/wrk/passive.yaml"
                    '''
                }
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop || true
                        docker rm zap || true
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo...'
            defectDojoPublisher(
                artifact: 'results/zap_xml_report.xml', 
                productName: 'Juice Shop', 
                scanType: 'ZAP Scan',
                engagementName: 'dawid.apolinarski@enp.pl'
            )
        }
    }
}