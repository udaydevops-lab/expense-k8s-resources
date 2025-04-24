pipeline {
    agent { label 'win-agent' }

    parameters {
        choice(name: 'BRANCH_TO_BUILD', choices: ['development'], description: 'Select Git branch')
        choice(name: 'ENVIRONMENT', choices: ['dev'], description: 'Choose the environment for deployment')
    }

    environment {
        WINDOWS_USER = 'administrator'
        WINDOWS_IP = '192.168.1.160'
        GIT_CREDENTIALS_ID = 'GitHub_User'
        GIT_URL = "https://github.com/SatraHIS/satra-his-staff-ts-next.git"
        NGINX_LOCATION = 'C:/nginx-1.23.1'
        NEXTJS_PATH = 'E:/sites/HIMS/satra-his-staff-ts-next'
        SCRIPT_PATH = 'C:/jenkins-agent/jenkins-scripts'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    def selectedBranch = params.BRANCH_TO_BUILD
                    checkout([$class: 'GitSCM',
                        branches: [[name: selectedBranch]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'CleanBeforeCheckout']],
                        userRemoteConfigs: [[
                            credentialsId: env.GIT_CREDENTIALS_ID,
                            url: env.GIT_URL
                        ]]
                    ])
                }
            }
        }

        // stage('Run PreDeploy Scripts') {
        //     steps {
        //         bat """
        //             cd ${SCRIPT_PATH}
        //             powershell -ExecutionPolicy Bypass -NoProfile -File pre-deploy.ps1
        //         """
        //     }
        // }

        // stage('Prepare Application') {
        //     steps {
        //             bat '''
        //                 cd ${NEXTJS_PATH}
        //                 powershell -Command 'Remove-Item -Path "${NEXTJS_PATH}/node_modules" -Force -Recurse -ErrorAction SilentlyContinue'
        //                 powershell -Command 'Remove-Item -Path "${NEXTJS_PATH}/.next" -Force -Recurse -ErrorAction SilentlyContinue'
        //             '''
        //     }
        // }

        // stage('Update Code and Install Dependencies') {
        //     steps {
        //         bat """
        //             xcopy /E /I /H /Y . "${NEXTJS_PATH}"
        //         """
        //         bat """
        //             cd "${NEXTJS_PATH}"
        //             npm install --force
        //         """
        //     }
        // }

        // stage('Build Application') {
        //     steps {
        //         script {
        //             catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
        //                 timeout(time: 3, unit: 'MINUTES') {
        //                     bat """
        //                         cd ${NEXTJS_PATH}
        //                         call npm run build:development
        //                     """
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('Run PostDeploy Scripts') {
        //     steps {
        //         bat """
        //             cd ${SCRIPT_PATH}
        //             powershell -ExecutionPolicy Bypass -NoProfile -File post-deploy.ps1
        //         """
        //     }
        // }

        stage('Check Application Status') {
            steps {
                script {
                    def timeoutSeconds = 180
                    def intervalSeconds = 5
                    def endTime = System.currentTimeMillis() + (timeoutSeconds * 1000)
                    def appUrl = 'https://front-next-dev.satragroup.in/'

                    echo "🔍 Checking if ${appUrl} is up..."

                    while (System.currentTimeMillis() < endTime) {
                        try {
                            def response = httpRequest(
                            url: appUrl,
                            validResponseCodes: '200:299',
                            timeout: 10
                        )
                            echo "✅ Application is UP with status: ${response.status}"
                            return // exit the stage successfully
                        } catch (e) {
                            echo "⏳ Application not up yet. Retrying in ${intervalSeconds} seconds..."
                            sleep time: intervalSeconds, unit: 'SECONDS'
                        }
                    }

                    // If reached here, it means it never succeeded within timeout
                    error("❌ Timeout: ${appUrl} did not respond successfully within ${timeoutSeconds / 60} minutes.")
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs for errors."
        }
    }
}
