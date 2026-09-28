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
               git branch: 'main',
                    url: 'https://github.com/khushboo12vishwakarma/fortify-multilang-demo'
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
                          -scan ^
                          > scancentral-submit.log 2>&1

                        type scancentral-submit.log
                        '''

                        def output = readFile(
                            file: 'scancentral-submit.log'
                        )

                        def matcher = output =~ /Submitted job and received token:\s*([0-9a-fA-F-]+)/

                        if (!matcher.find()) {
                            error(
                                'ScanCentral job token was not found.'
                            )
                        }

                        env.SCANCENTRAL_JOB_TOKEN = matcher.group(1)

                        echo '========================================'
                        echo 'SCANCENTRAL JOB CREATED'
                        echo '========================================'
                        echo "Job token captured successfully."
                    }
                }
            }
        }

        stage('Wait For Scan Completion') {
            steps {

                withCredentials([
                    string(
                        credentialsId: 'fortify-ssc-token',
                        variable: 'FORTIFY_TOKEN'
                    )
                ]) {

                    bat '''
                    echo ========================================
                    echo WAITING FOR SCANCENTRAL SCAN
                    echo ========================================

                    "%SCANCENTRAL_BIN%\\scancentral.bat" ^
                      -sscurl "%FORTIFY_SSC_URL%" ^
                      -ssctoken "%FORTIFY_TOKEN%" ^
                      status ^
                      -token "%SCANCENTRAL_JOB_TOKEN%" ^
                      -block-until scan ^
                      -bto 60 ^
                      -pi 30

                    if errorlevel 1 (
                        echo ScanCentral scan did not complete successfully.
                        exit /b 1
                    )
                    '''
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

                    bat '''
                    echo ========================================
                    echo RETRIEVING FORTIFY FPR
                    echo ========================================

                    "%SCANCENTRAL_BIN%\\scancentral.bat" ^
                      -sscurl "%FORTIFY_SSC_URL%" ^
                      -ssctoken "%FORTIFY_TOKEN%" ^
                      retrieve ^
                      -token "%SCANCENTRAL_JOB_TOKEN%" ^
                      -f "%WORKSPACE%\\FortifyMultiLangDemo.fpr" ^
                      -o

                    if errorlevel 1 (
                        echo Failed to retrieve FPR.
                        exit /b 1
                    )

                    echo ========================================
                    echo FPR CREATED
                    echo ========================================

                    dir "%WORKSPACE%\\FortifyMultiLangDemo.fpr"
                    '''
                }
            }
        }

        stage('Generate Fortify Report') {
    steps {
        echo '========================================'
        echo 'GENERATING FORTIFY HTML REPORT'
        echo '========================================'

        bat '''
            set "REPORT_GENERATOR=C:\\Program Files\\Fortify\\OpenText_Application_Security_Tools_25.4.0\\bin\\ReportGenerator.bat"
            set "FPR_FILE=%WORKSPACE%\\FortifyMultiLangDemo.fpr"
            set "REPORT_FILE=%WORKSPACE%\\FortifyMultiLangDemo-report.html"

            echo Report Generator:
            echo %REPORT_GENERATOR%

            echo.
            echo FPR:
            echo %FPR_FILE%

            echo.
            echo Checking files...

            if not exist "%REPORT_GENERATOR%" (
                echo ERROR: ReportGenerator.bat was not found.
                exit /b 1
            )

            if not exist "%FPR_FILE%" (
                echo ERROR: Fortify FPR was not found.
                exit /b 1
            )

            echo.
            echo Generating HTML report...

            call "%REPORT_GENERATOR%" ^
                -format html ^
                -f "%REPORT_FILE%" ^
                -source "%FPR_FILE%"

            if errorlevel 1 (
                echo ERROR: Failed to generate Fortify HTML report.
                exit /b 1
            )

            if not exist "%REPORT_FILE%" (
                echo ERROR: HTML report was not created.
                exit /b 1
            )

            echo.
            echo ========================================
            echo FORTIFY HTML REPORT CREATED
            echo ========================================
            dir "%REPORT_FILE%"
        '''
    }
}
    }

    post {

        always {
            echo '========================================'
            echo 'ARCHIVING FORTIFY RESULTS'
            echo '========================================'

            archiveArtifacts(
                artifacts: '*.fpr,*.html,scancentral-submit.log',
                allowEmptyArchive: true
            )
        }

        success {
            echo '========================================'
            echo 'FORTIFY PIPELINE COMPLETED'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'FORTIFY PIPELINE FAILED'
            echo '========================================'
        }
    }
}