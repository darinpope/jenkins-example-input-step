def firstName = null
pipeline {
  agent none
  stages {
    stage('input') {
      steps {
        script {
          firstName = input (
            message: 'What is your first name?', 
            ok: 'Submit', 
            parameters: [string(defaultValue: 'Dave', name: 'FIRST_NAME', trim: true)]
          )
        }
      }
    }
    stage('output') {
      agent any
      steps {
        script {
          echo "Good Morning, $firstName"
        }
        sh '''
          hostname
          cat /etc/redhat-release
        '''
      }
    }
  }
}