pipeline {
    agent any

    environment {
        // ===== AWS CONFIG =====
        BLUE_TG = "arn:aws:elasticloadbalancing:ap-northeast-3:735189764173:targetgroup/BLUE-TG/0403f43f313cb5e0"
        GREEN_TG = "arn:aws:elasticloadbalancing:ap-northeast-3:735189764173:targetgroup/Green-TG/55d8a8aff5446f00"
        LISTENER_ARN = "arn:aws:elasticloadbalancing:ap-northeast-3:735189764173:listener/app/blue-green-alb/92350f7357158c86/0a95985c95ef52f7"

        // ===== EC2 IPS =====
        BLUE_IP = "56.155.140.245"
        GREEN_IP = "56.155.45.60"
    }

    stages {

        // 🔹 STEP 1: Detect Active Environment
        stage('Select Target Environment') {
            steps {
                script {

                    def currentTG = sh(
                        script: """
                        aws elbv2 describe-listeners \
                        --listener-arn ${LISTENER_ARN} \
                        --query 'Listeners[0].DefaultActions[0].TargetGroupArn' \
                        --output text
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Current Target Group: ${currentTG}"

                    if (currentTG == env.BLUE_TG) {
                        env.TARGET_IP = env.GREEN_IP
                        env.TARGET_GROUP = env.GREEN_TG
                        env.DEPLOY_FILE = "green.html"
                        env.PREVIOUS_TG = env.BLUE_TG
                        env.NEXT_ENV = "GREEN"
                    } else {
                        env.TARGET_IP = env.BLUE_IP
                        env.TARGET_GROUP = env.BLUE_TG
                        env.DEPLOY_FILE = "blue.html"
                        env.PREVIOUS_TG = env.GREEN_TG
                        env.NEXT_ENV = "BLUE"
                    }

                    echo "Deploying to ${env.NEXT_ENV} environment (${env.TARGET_IP})"
                }
            }
        }

        // 🔹 STEP 2: Deploy to Inactive Environment
        stage('Deploy') {
            steps {
                script {
                    sh """
                    echo "Deploying ${DEPLOY_FILE} to ${TARGET_IP}"
                    scp -o StrictHostKeyChecking=no ${DEPLOY_FILE} ubuntu@${TARGET_IP}:/tmp/index.html
                    ssh -o StrictHostKeyChecking=no ubuntu@${TARGET_IP} "sudo mv /tmp/index.html /var/www/html/index.html"
                    """
                }
            }
        }

        // 🔹 STEP 3: Health Check
        stage('Health Check') {
            steps {
                script {
                    sleep 10

                    def status = sh(
                        script: "curl -s http://${TARGET_IP} | grep Version",
                        returnStatus: true
                    )

                    if (status != 0) {
                        error "Health check FAILED"
                    } else {
                        echo "Health check PASSED"
                    }
                }
            }
        }

        // 🔹 STEP 4: Switch Traffic
        stage('Switch Traffic') {
            steps {
                script {
                    sh """
                    echo "Switching traffic to ${NEXT_ENV}"
                    aws elbv2 modify-listener \
                    --listener-arn ${LISTENER_ARN} \
                    --default-actions Type=forward,TargetGroupArn=${TARGET_GROUP}
                    """
                }
            }
        }
    }

    // 🔁 POST ACTIONS
    post {

        // ✅ SUCCESS
        success {
            echo "Deployment SUCCESSFUL - Traffic switched"
        }

        // ❌ FAILURE → ROLLBACK
        failure {
            echo "Deployment FAILED - Rolling back..."

            sh """
            aws elbv2 modify-listener \
            --listener-arn ${LISTENER_ARN} \
            --default-actions Type=forward,TargetGroupArn=${PREVIOUS_TG}
            """
        }
    }
}
