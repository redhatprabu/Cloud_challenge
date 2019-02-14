pipeline {
  agent any

  environment {
    MAJOR_VERSION = 1
  }

  stages {
    stage('build') {
      steps {
        sh 'javac -d . src/*.java'
        sh 'echo Main-Class: Rectangulator > MANIFEST.MF'
        sh 'jar -cvmf MANIFEST.MF rectangle.jar *.class'
      }
      post {
        success {
          archiveArtifacts artifacts: 'rectangle.jar', fingerprint: true
        }
      }
    }
    stage('run') {
      steps {
        sh 'java -jar rectangle.jar 7 9'
      }
    }
    stage('Promote Development to Master') {
      when {
        branch 'development'
      }
      steps {
        echo "Stashing Local Changes"
        sh "git stash"
        echo "Checking Out Development"
        sh 'git checkout development'
        sh 'git pull origin'
        echo 'Checking Out Master'
        sh 'git checkout master'
        echo "Merging Development into Master"
        sh 'git merge development'
        echo "Git Push to Origin"
        sh 'git push origin master'
      }
      post {
        success {
          emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Promoted to Master",
            body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development Promoted to Master":</p>
            <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
            to: "websphereprabu@gmail.com"
          )
        }
      }
    }
    stage('Tagging the Release') {
      when {
        branch 'master'
      }
      steps {
        sh "git tag rectangle-${env.MAJOR_VERSION}.${BUILD_NUMBER}"
        sh "git push origin rectangle-${env.MAJOR_VERSION}.${BUILD_NUMBER}"
      }
      post {
        success {
          emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] NEW RELEASE",
            body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' NEW RELEASE":</p>
            <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
            to: "websphereprabu@gmail.com"
          )
        }
      }
    }
  }
}
