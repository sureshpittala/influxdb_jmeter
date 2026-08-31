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
        timeout(time: 10, unit: 'MINUTES') {
            bat '''
            set PATH=%JAVA_HOME%\\bin;%PATH%

            if not exist logs mkdir logs
            if not exist html mkdir html

            echo ==== JAVA VERSION ====
            java -version

            echo ==== RUNNING JMETER ====

            call "%JMETER_HOME%\\bin\\jmeter.bat" -n -t API_influx_grafana.jmx -l logs/results.jtl -e -o html/report -Jjmeterengine.force.system.exit=true
            echo JMeter Execution Completed
            '''            
        }
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

        cd /d C:\\practice\\AiPERF\\baselineintelligence

        echo Running Actuator Metrics Collector...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" actuator_metrics_collector.py

        echo Running Transaction History Writer...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" transaction_history_writer.py %RUN_ID%

        if errorlevel 1 (
            echo ERROR: Transaction History Writer Failed
            exit /b 1
        )

        echo Running Comparison Engine...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" transaction_comparison_report.py

        if errorlevel 1 (
            echo ERROR: Comparison Engine Failed
            exit /b 1
        )

        echo Running Transaction Comparison Matrix...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" transaction_comparison_matrix.py
        
        if errorlevel 1 (
            echo ERROR: Transaction Comparison Matrix Engine Failed
            exit /b 1
        )

        echo Running Variance Ranking...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" ai_variance_ranking.py

        if errorlevel 1 (
            echo ERROR: Variance Ranking Engine Failed
            exit /b 1
        )

        echo Running Executive Summary...
        "C:\\Users\\Suresh.Pittala\\AppData\\Local\\Programs\\Python\\Python312\\python.exe" ai_executive_summary.py

        if errorlevel 1 (
            echo ERROR: Executive Summary Engine Failed
            exit /b 1
        )
        
        echo Transaction History Writer Completed
        echo AiPERF History Processing Completed
        echo AiPERF Transaction Comparison Report Completed
        echo AiPERF Transaction Comparison Matrix Completed
        echo AiPERF Variance Ranking Completed
        echo AiPERF Running Executive Summary Completed
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
