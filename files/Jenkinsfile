pipeline {
      agent any

      tools {nodejs "NodeJS"}


      parameters{
          string(name: 'SPEC', defaultValue:"cypress/e2e/1-getting-started/todo.cy.js", description: "Enter the cypress script path that you want to execute")
          choice(name: 'BROWSER', choices:['electron', 'chrome', 'edge', 'firefox'], description: "Select the browser to be used in your cypress tests")
      }

      stages {
        stage('Run automated tests'){
            steps {
              echo "Running automated tests"
                sh 'npm prune'
                sh 'npm cache clean --force'
                sh 'npm i'
                sh 'npm install --save-dev mochawesome mochawesome-merge mochawesome-report-generator'
                sh 'npm run cypress:run'
            }
            post {
                success {
                    publishHTML (
                        target : [
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'mochawesome-report',
                            reportFiles: 'mochawesome.html',
                            reportName: 'My Reports',
                            reportTitles: 'The Report'])

                }
            }
        }
        stage('SonarQube analysis') {
          steps {
            script {
                      scannerHome = tool 'sonar-scanner';
                 }
            withSonarQubeEnv('QS_KITCHENSKIN_2220660') { // Replace this name by the one you setup in the SonarQube Jenkins configuration (step 2)
            sh "${scannerHome}/bin/sonar-scanner"
            }
          }
        }
        stage('JMeter Test') {
            steps {
                script {
                    // Path to the JMeter installation directory
                    def jmeterHome = '/usr/share/jmeter'

                    // Path to the JMeter test script
                    def jmeterScript = './testPlanHome.jmx'

                    // Execute JMeter test
                    sh "${jmeterHome}/bin/jmeter -n -t ${jmeterScript} -l result.jtl"
                }
            }
            post {
                always {
                    // Archive JTL result file
                    archiveArtifacts 'result.jtl'
                }
                success {
                    // Publish JMeter report using Performance plugin
                    perfReport filterRegex:'', sourceDataFiles: 'result.jtl'
                }
                failure {
                    script {
                        def errorCount = sh(script: '''awk -F ',' '\$8=="false"' result.jtl | wc -l''', returnStdout: true).trim()
                        if (Integer.parseInt(errorCount) > 10) {
                            error("Performance Test failed with more than 10 errors")
                        }
                    }
                }
            }
        }
        stage('Perform manual testing...'){
            steps {
                timeout(activity: true, time: 5) {
                    input 'Proceed to production?'
                }
           }
        }

        stage('Release to production') {
            steps {
            // similar procedure as in the 'Build/ Deploy to staging' stage, suppressed here for cost saving purposes
                echo "Deploying app in production environment"
           }
        }
    }
}