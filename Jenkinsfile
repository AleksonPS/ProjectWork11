pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Забираем изменения из GitHub
                //git 'https://github.com/AleksonPS/ProjectWork11'
                git url: 'https://github.com/AleksonPS/ProjectWork11.git', branch: 'main'
            }
        }
        stage('Build Nginx Container') {
            steps {
                // Поднимаем контейнер с nginx
                sh 'docker run -d --name nginx -p 9889:80 nginx'
                sh 'docker cp index.html nginx:/usr/share/nginx/html/index.html'
            }
        }
        stage('Test HTTP Response') {
            steps {
                // Проверяем код ответа
                script {
                    def httpResponse = sh(script: "curl -o /dev/null -s -w '%{http_code}' http://localhost:9889", returnStdout: true).trim()
                    if (httpResponse != '200') {
                        error "HTTP response is not 200, but ${httpResponse}"
                    }
                }
            }
        }
        stage('Compare MD5 Sums') {
            steps {
                // Сравниваем md5-суммы
                script {
                    def localMD5 = sh(script: "md5sum index.html | awk '{ print \$1 }'", returnStdout: true).trim()
                    def serverMD5 = sh(script: "curl -s http://localhost:9889/index.html | md5sum | awk '{ print \$1 }'", returnStdout: true).trim()
                    if (localMD5 != serverMD5) {
                        error "MD5 checksums do not match. Local: ${localMD5}, Server: ${serverMD5}"
                    }
                }
            }
        }
        stage('Cleanup') {
            steps {
                // Удаляем контейнер
                sh 'docker stop nginx'
                sh 'docker rm nginx'
            }
        }
    }
    post {
        failure {
            // Уведомления в случае ошибки
            mail to: 'alekson.ps@gmaile.com',
                subject: "Build Failed in Jenkins",
                body: "The build failed in stage ${STAGE_NAME}. Please check Jenkins logs."
        }
    }
}
