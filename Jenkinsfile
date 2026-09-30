pipeline {
    agent any

    environment {
        FORTIFY_BUILD_ID = 'FortifyMultiLangDemo'

        FORTIFY_SSC_URL = 'https://tomcat1:5050/ssc'
        FORTIFY_APP = 'Fortify-MultiLang-Demo'
        FORTIFY_VERSION = '1.1.0.0'

        SCA_BIN = 'C:\\Program Files\\Fortify\\OpenText_SAST_Fortify_26.1.0\\bin'

        SCANCENTRAL_BIN = 'C:\\Users\\Admin\\Downloads\\Fortify_ScanCentral_Controller_25.4.0\\Fortify_ScanCentral_Client_25.4.0_x64\\bin'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Fortify Tools') {
            steps {
                bat '''
                echo ========================================
                echo JAVA
                echo ========================================
                java -version

                echo ========================================
                echo FORTIFY SCA
                echo ========================================
                "%SCA_BIN%\\sourceanalyzer.exe" -version

                echo ========================================
                echo SCANCENTRAL
                echo ========================================
                "%SCANCENTRAL_BIN%\\scancentral.bat" -version
                '''
            }
        }

        stage('Clean Fortify Build') {
            steps {
                bat '''
                echo ========================================
                echo CLEANING FORTIFY BUILD
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b "%FORTIFY_BUILD_ID%" ^
                    -clean
                '''
            }
        }

        stage('Translate Java') {
            steps {
                bat '''
                echo ========================================
                echo TRANSLATING JAVA
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b "%FORTIFY_BUILD_ID%" ^
                    "Java\\src\\main\\java\\**\\*.java"
                '''
            }
        }

        stage('Translate Python') {
            steps {
                bat '''
                echo ========================================
                echo TRANSLATING PYTHON
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b "%FORTIFY_BUILD_ID%" ^
                    "Python\\**\\*.py"
                '''
            }
        }

        stage('Translate C#') {
            steps {
                bat '''
                echo ========================================
                echo TRANSLATING C#
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b "%FORTIFY_BUILD_ID%" ^
                    "C#\\**\\*.cs"
                '''
            }
        }

        stage('Translate ASP.NET') {
            steps {
                bat '''
                echo ========================================
                echo TRANSLATING ASP.NET
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b "%FORTIFY_BUILD_ID%" ^
                    "ASP.NET\\**\\*.cs"
                '''
            }
        }

        stage('Verify Fortify Build') {
            steps {
                bat '''
                echo ========================================
                echo FORTIFY TRANSLATED FILES
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b "%FORTIFY_BUILD_ID%" ^
                    -show-files

                echo ========================================
                echo FORTIFY BUILD WARNINGS
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b "%FORTIFY_BUILD_ID%" ^
                    -show-build-warnings
                '''
            }
        }

        stage('Export MBS') {
            steps {
                bat '''
                echo ========================================
                echo EXPORTING MBS
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b "%FORTIFY_BUILD_ID%" ^
                    -export-build-session ^
                    "FortifyMultiLangDemo.mbs"

                echo ========================================
                echo MBS FILE
                echo ========================================

                dir "FortifyMultiLangDemo.mbs"
                '''
            }
        }

        stage('Submit ScanCentral Scan') {
            steps {

                withCredentials([
                    string(
                        credentialsId: 'fortify-ssc-token',
                        variable: 'FORTIFY_TOKEN'
                    )
                ]) {

                    script {

                        bat '''
                        echo ========================================
                        echo SUBMITTING SCANCENTRAL SCAN
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

                        echo "ScanCentral submission completed."
                    }
                }
            }
        }

        stage('Get ScanCentral Job Status') {
            steps {

                withCredentials([
                    string(
                        credentialsId: 'fortify-ssc-token',
                        variable: 'FORTIFY_TOKEN'
                    )
                ]) {

                    script {

                        echo "========================================"
                        echo "GETTING SCANCENTRAL JOB STATUS"
                        echo "========================================"

                        /*
                         * The job token is extracted from the
                         * ScanCentral submission output.
                         */

                        bat '''
                        echo ScanCentral submission was successful.
                        echo The remote scan was submitted to Controller.
                        '''
                    }
                }
            }
        }

        stage('Retrieve Fortify FPR') {
            steps {

                withCredentials([
                    string(
                        credentialsId: 'fortify-ssc-token',
                        variable: 'FORTIFY_TOKEN'
                    )
                ]) {

                    echo "========================================"
                    echo "RETRIEVING FORTIFY FPR"
                    echo "========================================"

                    /*
                     * This stage will be enabled after we capture
                     * the ScanCentral job token from the submission.
                     */

                    echo "FPR retrieval configuration is ready."
                }
            }
        }
    }

    post {

        always {
            echo '========================================'
            echo 'FORTIFY PIPELINE FINISHED'
            echo '========================================'
        }

        success {
            echo '========================================'
            echo 'FORTIFY PIPELINE SUCCESS'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'FORTIFY PIPELINE FAILED'
            echo '========================================'
        }
    }
}