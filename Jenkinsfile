pipeline {
  agent any
  parameters {
    booleanParam description: 'Static_Check', name: 'Static_Check'
    booleanParam description: 'QA', name: 'QA'
    booleanParam description: 'Unit_Test', name: 'Unit_Test'
    string defaultValue: 'jotsnaram@gmail.com', description: 'Success_Email', name: 'Success_Email'
    string defaultValue: 'jotsnaram@gmail.com', description: 'Failure_Email', name: 'Failure_Email'
  }

  stages {

    stage("Is the run required") {
      steps {
        script {
          env.IS_HOLIDAY = isTodayHoliday()
        }
      }
    }

    stage("Build") {
      when {
        environment name: 'IS_HOLIDAY', value: "false"
      }
      steps {
        script {
          echo "Welcome...Build"
          def empJson = readJSON file: 'build.json'
          print empJson.employees
          empJson.employees.each {
            item ->
              dir('builds') {
                writeFile file: "${item.name}.txt", text: "${item.content}"
                echo "Name-- ${item.name}"
                echo "Content ${item.content}"
              }
          }
          if (fileExists('builds.zip')) {
            sh "rm builds.zip"
          }
          zip dir: 'builds', zipFile: 'builds.zip'

        }
      }
    }

    stage(" ") {
      parallel {

        stage("Quality") {
          stages {
            stage("Static_Check") {
              when {
                allOf {
                  environment name: 'IS_HOLIDAY', value: "false"
                  expression {
                    params.Static_Check == true
                  }
                }
              }
              steps {
                unzip dir: 'Static_Check', glob: '', zipFile: 'builds.zip'
              }

            }
            stage("QA") {
              when {
                allOf {
                  environment name: 'IS_HOLIDAY', value: "false"
                  expression {
                    params.QA == true
                  }
                }
              }
              steps {
                unzip dir: 'QA', glob: '', zipFile: 'builds.zip'
              }
            }

          }

        }

        stage("Unit_Test") {
          when {
            allOf {
              environment name: 'IS_HOLIDAY', value: "false"
              expression {
                params.Unit_Test == true
              }
            }

          }
          steps {
            unzip dir: 'Unit_Test', glob: '', zipFile: 'builds.zip'
          }
        }

      }

    }
    stage("Summary") {
      steps {
        script {
          if (env.IS_HOLIDAY == "false") {
            if (params.Unit_Test == true) {
              echo "Unit_test executed..."
              sh "ls Unit_Test"
            }
            if (params.QA == true) {
              echo "QA executed..."
              sh "ls QA"
            }
            if (params.Static_Check == true) {
              echo "Static_Check executed..."
              sh "ls Static_Check"
            }
          }
        }

      }
    }

  }
  post {
    success {
      echo "send email"
      mail body: 'Build success', subject: 'Build Success', to: params.Success_Email
    }
    failure {
      mail body: 'Build Failed', subject: 'Build Success', to: params.Failure_Email
    }
  }
}

def isTodayHoliday() {
  def dates = []
  def response = httpRequest 'https://calendarific.com/api/v2/holidays?&api_key=90e19de04681bc27e9492b478df436f43aed8c4c&country=IN&year=2021'
  println("Status: " + response.status)
  def jsonObj = readJSON text: response.content
  for (holiday in jsonObj.response.holidays) {
    dates.add(holiday.date.iso.split('T')[0])
  }
  def now = new Date()
  def currentDate = now.format("yyyy-MM-dd", TimeZone.getTimeZone('UTC'))
  return dates.contains(currentDate)
}
