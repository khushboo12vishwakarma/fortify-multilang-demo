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
                checkout scm
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

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b %FORTIFY_BUILD_ID% ^
                    -clean
                '''
            }
        }

        stage('Translate Java') {
            steps {
                bat '''
                echo ===== TRANSLATING JAVA =====

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b %FORTIFY_BUILD_ID% ^
                    "Java\\src\\main\\java\\**\\*.java"
                '''
            }
        }

        stage('Translate Python') {
            steps {
                bat '''
                echo ===== TRANSLATING PYTHON =====

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b %FORTIFY_BUILD_ID% ^
                    "Python\\**\\*.py"
                '''
            }
        }

        stage('Translate C#') {
            steps {
                bat '''
                echo ===== TRANSLATING C# =====

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b %FORTIFY_BUILD_ID% ^
                    "C#\\**\\*.cs"
                '''
            }
        }

        stage('Translate ASP.NET') {
            steps {
                bat '''
                echo ===== TRANSLATING ASP.NET =====

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b %FORTIFY_BUILD_ID% ^
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
                    -b %FORTIFY_BUILD_ID% ^
                    -show-files

                echo ========================================
                echo FORTIFY BUILD WARNINGS
                echo ========================================

                "%SCA_BIN%\\sourceanalyzer.exe" ^
                    -b %FORTIFY_BUILD_ID% ^
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
                    -b %FORTIFY_BUILD_ID% ^
                    -export-build-session "FortifyMultiLangDemo.mbs"

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

                        def submitOutput = readFile('scancentral-submit.log')

                        def matcher = submitOutput =~ /Submitted job and received token:\\s*([0-9a-fA-F-]+)/

                        if (!matcher.find()) {
                            error('Could not find ScanCentral job token.')
                        }

                        env.SCANCENTRAL_JOB_TOKEN = matcher.group(1)

                        echo "ScanCentral Job Token: ${env.SCANCENTRAL_JOB_TOKEN}"
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

                    echo ========================================
                    echo FPR FILE
                    echo ========================================

                    dir "%WORKSPACE%\\FortifyMultiLangDemo.fpr"
                    '''
                }
            }
        }

        stage('Generate Fortify Report') {
            steps {

                bat '''
                echo ========================================
                echo GENERATING FORTIFY REPORT
                echo ========================================

                "%SCA_BIN%\\ReportGenerator.bat" ^
                    -format html ^
                    -f "%WORKSPACE%\\FortifyMultiLangDemo-report.html" ^
                    -source "%WORKSPACE%\\FortifyMultiLangDemo.fpr"

                echo ========================================
                echo REPORT FILE
                echo ========================================

                dir "%WORKSPACE%\\FortifyMultiLangDemo-report.html"
                '''
            }
        }
    }

    post {

        always {
            echo '========================================'
            echo 'FORTIFY ARTIFACTS'
            echo '========================================'

            archiveArtifacts(
                artifacts: '*.fpr,*.html,scancentral-submit.log',
                allowEmptyArchive: true
            )
        }

        success {
            echo '========================================'
            echo 'FORTIFY PIPELINE COMPLETED SUCCESSFULLY'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'FORTIFY PIPELINE FAILED'
            echo '========================================'
        }
    }
}