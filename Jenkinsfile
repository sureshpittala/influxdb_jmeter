pipeline {
    agent any

    environment {
        JAVA_HOME = 'C:\\Program Files\\Eclipse Adoptium\\jdk-17.0.19.10-hotspot'
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

                "%JMETER_HOME%\\bin\\jmeter.bat" -n -t API_influx_grafana.jmx -l logs/results.jtl -e -o html/report -Jjmeterengine.force.system.exit=true
                '''
            }
        }

        stage('AiPERF History') {

    steps {

        script {
            env.RUN_ID = "RUN_${BUILD_NUMBER}_${new Date().format('yyyyMMdd_HHmmss')}"
        }

        echo "RUN_ID=${env.RUN_ID}"

        bat '''
        echo =====================================
        echo Build Number: %BUILD_NUMBER%
        echo Job Name: %JOB_NAME%
        echo Run ID: %RUN_ID%
        echo =====================================

        cd C:\\practice\\AiPERF\\baselineintelligence

        echo Running Actuator Metrics Collector...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" actuator_metrics_collector.py

        echo Running Execution History Writer...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" execution_history_writer.py

        echo Running Similar Execution Intelligence...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" similar_execution.py

        echo AiPERF History Processing Completed
        '''
    }
}
        stage('JMeter Timeout') {
    options {
        timeout(time: 5, unit: 'MINUTES')
    }
    steps {
        bat '''
        ...
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
