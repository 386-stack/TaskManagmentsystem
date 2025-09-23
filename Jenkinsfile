pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'          // Replace with your Jenkins JDK tool name
        maven 'MAVEN_HOME'       // Replace with your Jenkins Maven tool name
        nodejs 'NODE_HOME'     // Replace with your Jenkins NodeJS tool name
    }

    environment {
        BACKEND_DIR = 'BACKEND/TASKMANAGMENTSYSTEM'
        FRONTEND_DIR = 'FRONTEND/TASKMANAGMENTSYSTEMFRONTEND'
        GIT_REPO = 'https://github.com/386-stack/TaskManagmentsystem.git'  // Replace after pushing
        TOMCAT_URL = 'http://10.143.1.1:9090/manager/text/manager/text'
        TOMCAT_CREDENTIALS = credentials('tomcat-creds')
        BACKEND_WAR = 'backendtaskmanagementsystem.war'
        FRONTEND_WAR = 'frontendtaskmanagementsystem.war'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${env.GIT_REPO}"
            }
        }

        stage('Build Frontend (Vite)') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Package Frontend as WAR') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    bat '''
                    if not exist "%FRONTEND_WAR%_dir" mkdir "%FRONTEND_WAR%_dir\\WEB-INF"
                    xcopy /E /I /Y dist "%FRONTEND_WAR%_dir" 2>nul
                    cd /d "%FRONTEND_WAR%_dir"
                    jar -cvf ..\\..\\..\\%FRONTEND_WAR% * 2>nul
                    cd /d ..\\..\\..
                    '''
                }
            }
        }

        stage('Build Backend (Spring Boot WAR)') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    bat 'mvn clean package'
                    bat 'copy target\\backendtaskmanagementsystem.war ..\\..\\%BACKEND_WAR%'
                }
            }
        }

        stage('Deploy Backend to Tomcat') {
            steps {
                bat '''
                curl -u %TOMCAT_CREDENTIALS_USR%:%TOMCAT_CREDENTIALS_PSW% ^
                    --upload-file "%BACKEND_WAR%" ^
                    "%TOMCAT_URL%/deploy?path=/backendtaskmanagementsystem&update=true" || exit /b 1
                '''
            }
        }

        stage('Deploy Frontend to Tomcat') {
            steps {
                bat '''
                curl -u %TOMCAT_CREDENTIALS_USR%:%TOMCAT_CREDENTIALS_PSW% ^
                    --upload-file "%FRONTEND_WAR%" ^
                    "%TOMCAT_URL%/deploy?path=/frontendtaskmanagementsystem&update=true" || exit /b 1
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
            echo "Backend URL: http://10.143.1.1:9090/manager/text/backendtaskmanagementsystem"
            echo "Frontend URL: http://10.143.1.1:9090/manager/text/frontendtaskmanagementsystem"
        }

        failure {
            echo "❌ Deployment Failed! Check logs."
        }

        always {
            // Cleanup WAR files after build
            bat '''
            if exist "%BACKEND_WAR%" del /Q "%BACKEND_WAR%" 2>nul
            if exist "%FRONTEND_WAR%" del /Q "%FRONTEND_WAR%" 2>nul
            '''
        }
    }
}
