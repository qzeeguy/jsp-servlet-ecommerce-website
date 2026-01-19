pipeline {
    agent {
        docker {
            image 'my-docker-agent:latest'
            args '-v /root/.m2:/root/.m2'
        }
    }

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
    }

    stages {
        stage("Git checkout") {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: params.BRANCH_NAME]],
                        userRemoteConfigs: [[
                            url: 'https://github.com/qzeeguy/jsp-servlet-ecommerce-website.git',
                            credentialsId: 'github-cred'
                        ]]
                    ])
                }
            }
        }

        stage("Testing") {
            steps {
                echo "Test runs"
            }
        }

        stage("Build") {
            steps {
                sh "mvn clean package"
                sh "cp target/*.war ${WAR_FILE}"
            }
            post { success { archiveArtifacts artifacts: '**/target/*.war', fingerprint: true } }
        }

        
    }
        
        stage("Deploy to Dev") {
            when {
                expression { params.BRANCH_NAME == 'file/dev' }
            }
            steps {
                echo "Deploying to Dev environment..."
                // sh "scp ${WAR_FILE} ${DEV_HOST}:/home/ubuntu/apache-tomcat-9.0.113/webapps/"
            }
        }

        stage("Deploy to QA") {
            when {
                expression { params.BRANCH_NAME == 'file/qa' }
            }
            steps {
                echo "Deploying to QA environment..."
                // sh "scp ${WAR_FILE} ${QA_HOST}:/home/ubuntu/apache-tomcat-9.0.113/webapps/"
            }
        }

        stage("Deploy to Prod") {
            when {
                expression { params.BRANCH_NAME == 'file/prod' }
            }
            steps {
                input message: "Deploy to Production?", ok: "Deploy"
                echo "Deploying to Production environment..."
                // sh "scp ${WAR_FILE} ${PROD_HOST}:/home/ubuntu/apache-tomcat-9.0.113/webapps/"
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
