pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('JMeter') {
      steps {
        bat '''
        set JAVA_HOME=C:\\Coforge Software\\jdk-25_windows-x64_bin\\jdk-25.0.35
        set PATH=%JAVA_HOME%\\bin;%PATH%
        if exist logs rmdir /s /q logs
        if exist html rmdir /s /q html
        mkdir logs
        mkdir html
        mkdir html\\report
        "C:\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter.bat" -n -t API.jmx -l logs/results.jtl -e -o html/report
        '''
      }
    }
    stage('Reports') {
      steps {
        perfReport sourceDataFiles: 'logs/results.jtl'
        publishHTML(
          target: [
            reportDir: 'html/report',
            reportFiles: 'index.html',
            reportName: 'JMeter HTML Report',
            keepAll: true,
            alwaysLinkToLastBuild: true,
            allowMissing: false
            ]
          )
      }
    }
  }
  post {
    always {
      archiveArtifacts 'logs/results.jtl, html/report/**'
    }
  }
}
