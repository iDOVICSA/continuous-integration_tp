pipeline {
  agent any
  stages {
    stage('Build') {
      post {
        failure {
          steps() {
            mail(subject: 'Build Project', body: 'The build failed', cc: 'hb_chergui@esi.dz')
          }

        }

      }
      steps {
        bat 'D:\\programs\\gradle-5.6-bin\\gradle-5.6\\bin\\gradle.bat build'
        bat 'D:\\programs\\gradle-5.6-bin\\gradle-5.6\\bin\\gradle.bat javadoc'
        archiveArtifacts(artifacts: 'build/libs/*.jar', allowEmptyArchive: true)
        archiveArtifacts 'build/reports/tests/test/**/*.*'
      }
    }

    stage('Mail notification') {
      steps {
        mail(subject: 'Build Project', body: 'The build was successful', cc: 'hb_chergui@esi.dz')
      }
    }

    stage('Code Analysis') {
      parallel {
        stage('Code Analysis') {
          steps {
            withSonarQubeEnv('Sonar') {
              bat 'D:\\\\programs\\\\gradle-5.6-bin\\\\gradle-5.6\\\\bin\\\\gradle.bat sonarqube'
            }

            script {
              timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
              def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
              if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
            }
          }

        }
      }

      stage('Test Reporting') {
        steps {
          cucumber 'reports/example-report.json'
        }
      }

    }
  }

  stage('Deploy') {
    steps {
      bat 'D:\\programs\\gradle-5.6-bin\\gradle-5.6\\bin\\gradle.bat publish'
    }
  }

}
}