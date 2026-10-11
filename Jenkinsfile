pipeline {
    agent { label 'podman' }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        REGISTRY = 'registry.assign.mabeza.300.ops:5000'
        IMAGE    = 'bind-secondary'
        TRIVY    = '/home/ubuntu/bin/trivy'
    }

    stages {
        stage('Build') {
            steps {
                sh 'hostname -f'
                sh 'podman build -t $REGISTRY/$IMAGE:$BUILD_NUMBER .'
            }
        }
        stage('Scan') {
            steps {
                sh 'podman save -o scan-image.tar $REGISTRY/$IMAGE:$BUILD_NUMBER'
                sh '$TRIVY image --input scan-image.tar --severity HIGH,CRITICAL --scanners vuln --exit-code 0 --output trivy-report.txt'
                sh 'cat trivy-report.txt'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
                    sh 'rm -f scan-image.tar'
                }
            }
        }
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'registry-login',
                                                  usernameVariable: 'REG_USER',
                                                  passwordVariable: 'REG_PASS')]) {
                    sh 'echo "$REG_PASS" | podman login "$REGISTRY" -u "$REG_USER" --password-stdin'
                }
                sh 'podman push $REGISTRY/$IMAGE:$BUILD_NUMBER'
            }
        }
    }

    post {
        always {
            sh 'podman logout $REGISTRY || true'
        }
    }
}
