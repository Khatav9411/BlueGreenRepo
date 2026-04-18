pipeline {
    agent any

    environment {
        BLUE_TG = "arn:aws:elasticloadbalancing:ap-northeast-3:735189764173:targetgroup/BLUE-TG/0403f43f313cb5e0"
        GREEN_TG = "arn:aws:elasticloadbalancing:ap-northeast-3:735189764173:targetgroup/Green-TG/55d8a8aff5446f00"
        LISTENER_ARN = "arn:aws:elasticloadbalancing:ap-northeast-3:735189764173:listener/app/blue-green-alb/92350f7357158c86/0a95985c95ef52f7"

        BLUE_IP = "56.155.140.245"
        GREEN_IP = "56.155.45.60"

        ACTIVE_ENV = "blue"
    }

    stages {

        // stage('Clone Code') {
            // steps {
                // git branch: 'main', url: 'https://github.com/Khatav9411/BlueGreenRepo.git'
            // }
        // }

        stage('Select Target Environment') {
            steps {
                script {
                    if (env.ACTIVE_ENV == "blue") {
                        env.TARGET_IP = env.GREEN_IP
                        env.TARGET_GROUP = env.GREEN_TG
                    } else {
                        env.TARGET_IP = env.BLUE_IP
                        env.TARGET_GROUP = env.BLUE_TG
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.TARGET_ENV == "GREEN") {
                        sh '''
                        scp -o StrictHostKeyChecking=no green.html ubuntu@<GREEN_IP>:/tmp/index.html
                        ssh ubuntu@<GREEN_IP> "sudo mv /tmp/index.html /var/www/html/index.html"
                        '''
                    } else {
                        sh '''
                        scp -o StrictHostKeyChecking=no blue.html ubuntu@<BLUE_IP>:/tmp/index.html
                        ssh ubuntu@<BLUE_IP> "sudo mv /tmp/index.html /var/www/html/index.html"
                        '''
                    }
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    sleep 10
                    def status = sh(
                        script: "curl -s http://${TARGET_IP} | grep Version",
                        returnStatus: true
                    )

                    if (status != 0) {
                        error "Health check failed"
                    }
                }
            }
        }

        stage('Switch Traffic') {
            steps {
                sh """
                aws elbv2 modify-listener \
                --listener-arn ${LISTENER_ARN} \
                --default-actions Type=forward,TargetGroupArn=${TARGET_GROUP}
                """
            }
        }
    }

    post {
        failure {
            echo "Rollback triggered"

            sh """
            aws elbv2 modify-listener \
            --listener-arn ${LISTENER_ARN} \
            --default-actions Type=forward,TargetGroupArn=${BLUE_TG}
            """
        }
    }
}
