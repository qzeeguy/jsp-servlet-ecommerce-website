pipeline {
    agent { label 'master' }

    parameters {
        string(
            name: "BRANCH_NAME",
            defaultValue: "file/dev",
            description: "Specify the branch name to deploy"
        )
    }

    environment {
        DEV_HOST  = "ubuntu@<DEV_EC2_IP>"
        QA_HOST   = "ubuntu@<QA_EC2_IP>"
        PROD_HOST = "ubuntu@<PROD_EC2_IP>"
        WAR_FILE  = "target/jsp-servlet-ecommerce-website.war"
        SSH_KEY   = ""
    }

    stages {

        stage("Git Checkout") {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: params.BRANCH_NAME]],
                    userRemoteConfigs: [[
                        url: 'https://github.com/qzeeguy/jsp-servlet-ecommerce-website.git',
                        credentialsId: 'sirvlet-cred'
                    ]]
                ])
            }
        }

        stage("Testing") {
            steps {
                echo "Test runs"
            }
        }

        stage("Build") {
            steps {
                sh '''
                    mvn clean package
                    ls -l target
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        stage("Deploy to Dev") {
            when {
                expression { params.BRANCH_NAME == 'file/dev' }
            }
            steps {
                echo "Deploying to Dev environment..."
                sshagent(credentialsId: ['jenkins-deploy-key']) {
                    sh """
                        set -x

                        ssh ${DEV_HOST} 'rm -rf /home/ubuntu/apache-tomcat-9.0.113/webapps/test-1.0-SNAPSHOT*'

                        scp ${env.WAR_FILE} ${DEV_HOST}:/home/ubuntu/apache-tomcat-9.0.113/webapps/

                        # Optionally restart Tomcat to ensure deployment
                        ssh ${DEV_HOST} '/home/ubuntu/apache-tomcat-9.0.113/bin/shutdown.sh || true'
                        ssh ${DEV_HOST} '/home/ubuntu/apache-tomcat-9.0.113/bin/startup.sh'
                    """
                }
            }
        }

        stage("Deploy to QA") {
            when {
                expression { params.BRANCH_NAME == 'file/qa' }
            }
            steps {
                echo "Deploying to QA environment..."
            }
        }

        stage("Deploy to Prod") {
            when {
                expression { params.BRANCH_NAME == 'file/prod' }
            }
            steps {
                input message: "Deploy to Production?", ok: "Deploy"
                echo "Deploying to Production environment..."
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
            slackSend(
                channel: '#cicd_monitoring',
                message: "✅ *SUCCESS* | Job: `${env.JOB_NAME}` | Build: #${env.BUILD_NUMBER} | Branch: `${params.BRANCH_NAME}`"
            )
        }
        failure {
            echo "Pipeline failed. Check logs."
            slackSend(
                channel: '#cicd_monitoring',
                message: "❌ *FAILED* | Job: `${env.JOB_NAME}` | Build: #${env.BUILD_NUMBER} | Branch: `${params.BRANCH_NAME}`"
            )
        }
        aborted {
            slackSend(
                channel: '#cicd_monitoring',
                message: "⚠️ *ABORTED* | Job: `${env.JOB_NAME}` | Build: #${env.BUILD_NUMBER} | Branch: `${params.BRANCH_NAME}`"
            )
        }
    }
}
