pipeline {
  agent any
  parameters {
    booleanParam description: 'Perform Static Check?', name: 'Static_Check'
    booleanParam description: 'Do you wanna run in QA environment?', name: 'QA'
    booleanParam description: 'Do you wanna perform unit testing?', name: 'Unit_Test'
    string defaultValue: 'jotsnaram@gmail.com', description: 'Mention emails to send success status', name: 'Success_Email'
    string defaultValue: 'jotsnaram@gmail.com', description: 'Mention emails to send failure status', name: 'Failure_Email'
  }
  environment{
      HOLIDAY_API_URL = "https://calendarific.com/api/v2/holidays?&api_key=90e19de04681bc27e9492b478df436f43aed8c4c&country=IN&year=2022"
  }

  stages {

    stage("Is the run required") {
      steps {
        script {
          env.IS_HOLIDAY = isHoliday()
        }
      }
    }

    stage("Build") {
      when {
        environment name: 'IS_HOLIDAY', value: "false"
      }
      steps {
        script {
          echo "Build Stage started"
          def empJson = readJSON file: 'build.json'
          print empJson.employees
          empJson.employees.each {
            item ->
              dir('builds') {
                writeFile file: "${item.name}.txt", text: "${item.content}"
              }
          }
          if (fileExists('builds.zip')) {
            sh "rm builds.zip"
          }
          zip dir: 'builds', zipFile: 'builds.zip'

        }
      }
    }

    stage("Quality & QA & Unit Test") {
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
            }
            if (params.QA == true) {
              echo "QA executed..."
            }
            if (params.Static_Check == true) {
              echo "Static_Check executed..."
            }
          }
        }

      }
    }

  }
  post {
    success {
      echo "Success email"
      //mail body: 'Build success', subject: 'Build Success', to: params.Success_Email
    }
    failure {
        echo "Failure email"
      // mail body: 'Build Failed', subject: 'Build Success', to: params.Failure_Email
    }
  }
}

def isHoliday() {
  def response = httpRequest url: env.HOLIDAY_API_URL, httpMode: 'GET'
  println("Status: " + response.status)
  def jsonObj = readJSON text: response.content
  def holidays = []
  for (holiday in jsonObj.response.holidays) {
    // few dates are in this format "2021-09-23T00:51:11+05:30", so splitting it 
    holidays.add(holiday.date.iso.split('T')[0])
  }
  def today = new Date()
  today = today.format("yyyy-MM-dd", TimeZone.getTimeZone('UTC'))

  return holidays.contains(today)
}
