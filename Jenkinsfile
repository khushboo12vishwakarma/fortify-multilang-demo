pipeline {
    agent any

    environment {
        FORTIFY_BUILD_ID = 'FortifyMultiLangDemo'
        FORTIFY_SSC_URL  = 'https://tomcat1:5050/ssc'
        FORTIFY_APP      = 'Fortify-MultiLang-Demo'
        FORTIFY_VERSION  = '1.1.0.0'

        SCA_BIN = 'C:\\Program Files\\Fortify\\OpenText_SAST_Fortify_26.1.0\\bin'

        SCANCENTRAL_BIN = 'C:\\Users\\Admin\\Downloads\\Fortify_ScanCentral_Controller_25.4.0\\Fortify_ScanCentral_Client_25.4.0_x64\\bin'
    }

    stages {

        stage('Checkout') {
            steps {
                 git branch: 'main',
                    url: 'https://github.com/khushboo12vishwakarma/fortify-multilang-demo'
            }
        }

        stage('Verify Fortify Tools') {
            steps {
                bat '''
                echo ===== JAVA =====
                java -version

                echo ===== SOURCEANALYZER =====
                "%SCA_BIN%\\sourceanalyzer.exe" -version

                echo ===== SCANCENTRAL =====
                "%SCANCENTRAL_BIN%\\scancentral.bat" -version
                '''
            }
        }

        stage('Clean Fortify Build') {
            steps {
                bat '''
                echo ===== CLEAN FORTIFY BUILD =====

                "%SCA_BIN%\\sourceanalyzer.exe" -b %FORTIFY_BUILD_ID% -clean
                '''
            }
        }

        stage('Translate Java') {
            steps {
                bat '''
                echo ===== TRANSLATING JAVA =====

                "%SCA_BIN%\\sourceanalyzer.exe" -b %FORTIFY_BUILD_ID% "Java\\src\\main\\java\\**\\*.java"
                '''
            }
        }

        stage('Translate Python') {
            steps {
                bat '''
                echo ===== TRANSLATING PYTHON =====

                "%SCA_BIN%\\sourceanalyzer.exe" -b %FORTIFY_BUILD_ID% "Python\\**\\*.py"
                '''
            }
        }

        stage('Translate C#') {
            steps {
                bat '''
                echo ===== TRANSLATING C# =====

                "%SCA_BIN%\\sourceanalyzer.exe" -b %FORTIFY_BUILD_ID% "C#\\**\\*.cs"
                '''
            }
        }

        stage('Translate ASP.NET') {
            steps {
                bat '''
                echo ===== TRANSLATING ASP.NET =====

                "%SCA_BIN%\\sourceanalyzer.exe" -b %FORTIFY_BUILD_ID% "ASP.NET\\**\\*.cs"
                '''
            }
        }

        stage('Verify Fortify Build') {
            steps {
                bat '''
                echo ========================================
                echo FORTIFY TRANSLATED FILES
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" -b %FORTIFY_BUILD_ID% -show-files

                echo ========================================
                echo FORTIFY BUILD WARNINGS
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" -b %FORTIFY_BUILD_ID% -show-build-warnings
                '''
            }
        }

        stage('Export MBS') {
            steps {
                bat '''
                echo ========================================
                echo EXPORTING MBS
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" -b %FORTIFY_BUILD_ID% -export-build-session "FortifyMultiLangDemo.mbs"

                dir "FortifyMultiLangDemo.mbs"
                '''
            }
        }

        stage('ScanCentral Scan') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'fortify-ssc-token',
                        variable: 'FORTIFY_TOKEN'
                    )
                ]) {
                    bat '''
                    echo ========================================
                    echo STARTING SCANCENTRAL SCAN
                    echo ========================================

                    "%SCANCENTRAL_BIN%\\scancentral.bat" ^
                      -sscurl "%FORTIFY_SSC_URL%" ^
                      -ssctoken "%FORTIFY_TOKEN%" ^
                      start -upload ^
                      --application "%FORTIFY_APP%" ^
                      --application-version "%FORTIFY_VERSION%" ^
                      -mbs "%WORKSPACE%\\FortifyMultiLangDemo.mbs" ^
                      -uptoken "%FORTIFY_TOKEN%" ^
                      -scan
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '===== FORTIFY PIPELINE COMPLETED ====='
        }

        failure {
            echo '===== FORTIFY PIPELINE FAILED ====='
        }
    }
}