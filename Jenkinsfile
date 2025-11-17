pipeline {
    agent any
    
    tools {
        nodejs 'NODEJS'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '=== Checkout del código ==='
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo '=== Instalando dependencias ==='
                sh 'npm ci --legacy-peer-deps' 
            }
        }
        
        stage('Tests & Coverage') {
            steps {
                echo '=== Ejecutando tests con cobertura ==='
                sh '''
                    export CHROME_BIN=/usr/bin/chromium
                    npm run test -- --watch=false --code-coverage --browsers=ChromeHeadlessCI
                '''
            }
            post {
                always {
                    script {
                        if (fileExists('coverage/index.html')) {
                            publishHTML(target: [
                                reportDir: 'coverage',
                                reportFiles: 'index.html',
                                reportName: 'Coverage Report',
                                keepAll: true,
                                alwaysLinkToLastBuild: true,
                                allowMissing: false
                            ])
                        } else {
                            echo '⚠️ No se generó reporte de cobertura'
                        }
                    }
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo '=== Análisis de SonarQube ==='
                script {
                    def scannerHome = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=petclinic-angular \
                            -Dsonar.sources=src \
                            -Dsonar.tests=src \
                            -Dsonar.test.inclusions=**/*.spec.ts \
                            -Dsonar.exclusions=**/*.spec.ts,node_modules/**,dist/** \
                            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                        """  
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo '=== Esperando Quality Gate ==='
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline ejecutado correctamente'
        }
        failure {
            echo '❌ Pipeline falló'
        }
        always {
            echo '=== Limpieza ==='
            deleteDir()
        }
    }
}