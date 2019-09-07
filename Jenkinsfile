#!groovy

//
// Mandatory Build Parameters
//
//   LABEL e.g. ndw1757
//   EPICS_HOST_ARCH   e.g. windows-x64
//
// Optional Build Parameters
//
//   CLEAN set to YES to clean after build
//

def genstep(step_name, params, env) {
       script{
       if (params.CLEAN != null && params.CLEAN == "YES") {
           env.CLEAN_BUILD = "clean"
       } else {
           env.CLEAN_BUILD = ""
       }
       if (params.ZIPNAME != null) {
            env.ZIPNAME = params.ZIPNAME
       } else {
            env.ZIPNAME = "\"\""
       }
	   }
       echo "Environment set, executing batch file"
       bat """
            set BUILD_NUMBER=${env.BUILD_NUMBER}
            set JOB_NAME=${env.JOB_NAME}
            set RELEASE=NO
       """
}

pipeline {

  // agent defines where the pipeline will run.
  agent {
    label {
      label("${params.LABEL}")
    }
  }

  environment {
      NODE = "${env.NODE_NAME}"
      ELOCK = "epics_${NODE}"
  }

  // The options directive is for configuration that applies to the whole job.
  options {
    buildDiscarder(logRotator(numToKeepStr:'5', daysToKeepStr: '7'))
    disableConcurrentBuilds()
    timestamps()
    // as we "checkout scm" as a stage, we do not need to do it here too
    skipDefaultCheckout(true)
  }

  stages {
    stage("Checkout") {
      steps {
        timeout(time: 2, unit: 'HOURS') {
          checkout scm
        }
      }
    }

    stage("Dependencies") {
        steps {
          echo "Installing local genie python"
          timeout(time: 1, unit: 'HOURS') {
          }
        }
    }
    
    stage("Build and Test") {
        steps {
            // lock a shared node specific resource in case two different EPICS builds try to run
            echo "Building"
            lock(resource: ELOCK, inversePrecedence: true) {
              timeout(time: 16, unit: 'HOURS') {
                echo "Branch: ${env.BRANCH_NAME}"
                echo "Build Number: ${env.BUILD_NUMBER}"
				genstep("BUILD", params, env)
				genstep("TEST", params, env)
              }
            }
        }
    }

    stage("Deploy") {
        steps {
          echo "Deploying"
            timeout(time: 6, unit: 'HOURS') {
		    genstep("DEPLOY", params, env)
          }
        }
    }

    stage("Symbols") {
      steps {
        echo "Debug Symbols and EPICS utils"
         // need a jenkins wide resource as epics builds on any node can write to same network directory/file
         lock(resource: 'epics_symbols', inversePrecedence: true) {
           timeout(time: 1, unit: 'HOURS') {
		    genstep("SYMBOLS", params, env)
         }
        }
    }
  }

  post {
    always {
        xunit (
            thresholds: [ skipped(failureThreshold: '0'), failed(failureThreshold: '0') ],
            tools: [ GoogleTest(pattern: '**/test-reports/TEST-*.xml') ]
            )
    }
    changed {
      step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'icp-buildserver@lists.isis.rl.ac.uk', sendToIndividuals: true])
    }
    failure {
      step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'icp-buildserver@lists.isis.rl.ac.uk', sendToIndividuals: true])
      slackSend channel: '#jenkins',
                  color: 'good',
                  message: "The pipeline ${currentBuild.fullDisplayName} failed."
    }
    unstable {
      step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: 'icp-buildserver@lists.isis.rl.ac.uk', sendToIndividuals: true])
    }
    cleanup {
        echo "Cleaning"
        timeout(time: 2, unit: 'HOURS') {
		  genstep("CLEAN", params, env)
          bat """
                  set WORKWIN=%WORKSPACE:/=\\%
                  if "${env.CLEAN_BUILD}" == "clean" (
                  )
          """
        }
    }
  }  
}

