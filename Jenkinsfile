pipeline {
    agent any
    tools {
        jdk 'JAVA_HOME'
        maven 'MAVEN_HOME'
    }
    environment {
        BACKEND_DIR = 'BACKEND/TASKMANAGMENTSYSTEM'
        FRONTEND_DIR = 'FRONTEND'
        BACKEND_WAR = 'backendtaskmanagementsystem.war'
        FRONTEND_WAR = 'frontendtaskmanagementsystem.war'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/386-stack/TaskManagmentsystem.git'
            }
        }

        stage('Build Frontend') {
            steps {
                script {
                    def nodeHome = tool name: 'NODE_HOME', type: 'NodeJS'
                    env.PATH = "${nodeHome};${env.PATH}"
                }
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

        stage('Build Backend') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    bat 'mvn clean package'
                    bat 'copy target\\%BACKEND_WAR% ..\\..\\%BACKEND_WAR%'
                }
            }
        }

        stage('Deploy Backend to Tomcat') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-creds', path: '/backendtaskmanagementsystem', url: 'http://localhost:9090')],
                       contextPath: '/backendtaskmanagementsystem',
                       war: "${env.BACKEND_WAR}"
            }
        }

        stage('Deploy Frontend to Tomcat') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-creds', path: '/frontendtaskmanagementsystem', url: 'http://localhost:9090')],
                       contextPath: '/frontendtaskmanagementsystem',
                       war: "${env.FRONTEND_WAR}"
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
        always {
            bat '''
            if exist "%BACKEND_WAR%" del /Q "%BACKEND_WAR%" 2>nul
            if exist "%FRONTEND_WAR%" del /Q "%FRONTEND_WAR%" 2>nul
            '''
        }
    }
}
