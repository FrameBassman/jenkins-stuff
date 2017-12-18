import hudson.tasks.test.AbstractTestResultAction
import groovy.json.JsonSlurper

def resultMessage = ""
def containersHaveBeenRun = false;

// variables for omit running some stages
def skipUnitTests = false;
def skipUITests = false;
def skipAPITests = false;

pipeline {
  agent {
    label 'master'
  }
  options {
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
  }
  stages {

    stage('Define build steps') {
      steps {
        script {
          echo "${env.CHANGE_TARGET}"
        }
      }
    }

  }
}

def notifySlack (String buildResult, String msg) {
  def colorCode = ""
  echo "Build result: " + buildResult

  if ( !env.CHANGE_TARGET ) { 
    slackChannel = env.SLACK_CHANNEL_OPEN 
  }
  testStatus = "${env.BRANCH_NAME} » ${env.BUILD_NUMBER} » <${env.BUILD_URL}|Open>\n"
  if ( buildResult == null ) {
    // 'GREEN'
    colorCode = '#69D89B'
    testStatus = testStatus + "Build status: SUCCESSFUL"
  } else if ( buildResult == 'UNSTABLE' ) {
    // 'YELLOW'
    colorCode = '#FFFF00'
    testStatus = testStatus + "Build status: UNSTABLE"
  } else  {
    // 'RED'
    colorCode = '#FF0000'
    testStatus = testStatus + "Build status: " + buildResult + "\n" + msg
  }

  slackSend color: colorCode, message: testStatus;
}

@NonCPS
def getTestSummary () {
  def testResultAction = currentBuild.rawBuild.getAction(AbstractTestResultAction.class)
  def summary = ""

  if (testResultAction != null) {
    def total = testResultAction.getTotalCount()
    def failed = testResultAction.getFailCount()
    def skipped = testResultAction.getSkipCount()

    summary = "Test results:\n\t"
    summary = summary + ("Passed: " + (total - failed - skipped))
    summary = summary + (", Failed: " + failed)
    summary = summary + (", Skipped: " + skipped)
    summaty = summary + ("\nFailed tests: " + failed)

  } else {
    summary = "No tests found"
  }

  return summary
}