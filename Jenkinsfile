pipeline {
    agent any

    tools {
        maven "M3"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/jws3873/deploy-test', branch: 'main'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def jarFile = 'target/shortenurlservice-0.0.1-SNAPSHOT.jar'
                    def serverIp = '15.164.231.166'
                    def deployPath = '/home/ubuntu'
                    def runAppCommand = "nohup java -jar $deployPath/shortenurlservice-0.0.1-SNAPSHOT.jar > $deployPath/app.log 2>&1 &"
                    def checkLogCommand = "grep -q 'Started ShortenurlserviceApplication in' $deployPath/app.log"
                    
                    // 기존 애플리케이션 종료
                    sshagent(['deploy_ssh_key']) {
                        sh script: "ssh ubuntu@$serverIp 'pgrep -f shortenurlservice-0.0.1-SNAPSHOT.jar && pkill -f shortenurlservice-0.0.1-SNAPSHOT.jar || echo \"No process found\"'", returnStatus: true
                    }
                    // 서버에 파일을 SCP로 전송
                    sh "scp -o StrictHostKeyChecking=no $jarFile ubuntu@$serverIp:$deployPath/"
                    
                    // 원격 서버에서 애플리케이션 비동기 실행
                    sshagent(['deploy_ssh_key']) {
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@$serverIp '$runAppCommand'"
                        sleep 20 // 애플리케이션이 시작될 시간을 제공합니다.
                        
                        // 로그 파일을 확인하여 애플리케이션 실행 확인
                        int result = sh script: "ssh -o StrictHostKeyChecking=no ubuntu@$serverIp '$checkLogCommand'", returnStatus: true
                        if (result == 0) {
                            echo 'Deployment was successful.'
                        } else {
                            error 'Deployment failed.'
                        }
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Deployment encountered an error.'
        }
    }
}