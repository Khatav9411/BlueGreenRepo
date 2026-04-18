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

        stage('Select Target Environment') {
            steps {
                script {
                    if (env.ACTIVE_ENV == "blue") {
                        env.TARGET_IP = env.GREEN_IP
                        env.TARGET_GROUP = env.GREEN_TG
                        env.DEPLOY_FILE = "green.html"
                        env.NEXT_ENV = "green"
                    } else {
                        env.TARGET_IP = env.BLUE_IP
                        env.TARGET_GROUP = env.BLUE_TG
                        env.DEPLOY_FILE = "blue.html"
                        env.NEXT_ENV = "blue"
                    }

                    echo "Deploying to ${env.NEXT_ENV} server (${env.TARGET_IP})"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                    echo "Copying ${DEPLOY_FILE} to ${TARGET_IP}"
                    scp -o StrictHostKeyChecking=no ${DEPLOY_FILE} ubuntu@${TARGET_IP}:/tmp/index.html
                    ssh -o StrictHostKeyChecking=no ubuntu@${TARGET_IP} "sudo mv /tmp/index.html /var/www/html/index.html"
                    """
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
                    } else {
                        echo "Health check passed"
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
        success {
            script {
                echo "Deployment successful!"

                if (env.ACTIVE_ENV == "blue") {
                    env.ACTIVE_ENV = "green"
                } else {
                    env.ACTIVE_ENV = "blue"
                }
            }
        }

        failure {
            echo "Rollback triggered!"

            sh """
            aws elbv2 modify-listener \
            --listener-arn ${LISTENER_ARN} \
            --default-actions Type=forward,TargetGroupArn=${BLUE_TG}
            """
        }
    }
}
