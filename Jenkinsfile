pipeline {
    agent any

    environment {
        JAVA_HOME = 'C:\\Coforge Software\\jdk-25_windows-x64_bin\\jdk-25.0.3'
        JMETER_HOME = 'C:\\jmeter\\apache-jmeter-5.6.3'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Clean Workspace') {
            steps {
                bat '''
                if exist logs rmdir /s /q logs
                if exist html rmdir /s /q html
                '''
            }
        }

        stage('JMeter Execution') {
            steps {
                bat '''
                set PATH=%JAVA_HOME%\\bin;%PATH%
                if not exist logs mkdir logs
                if not exist html mkdir html

                echo ==== JAVA VERSION ====
                java -version

                echo ==== RUNNING JMETER ====

                "%JMETER_HOME%\\bin\\jmeter.bat" -n -t API.jmx -l logs/results.jtl -e -o html/report -Jjmeterengine.force.system.exit=true
                '''
            }
        }

        stage('Publish Reports') {
            steps {
                perfReport sourceDataFiles: 'logs/results.jtl'

                publishHTML(target: [
                    reportDir: 'html/report',
                    reportFiles: 'index.html',
                    reportName: 'JMeter HTML Report',
                    keepAll: true,
                    alwaysLinkToLastBuild: true,
                    allowMissing: false
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'logs/results.jtl, html/report/**', fingerprint: true
        }
    }
}
