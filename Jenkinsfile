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
    timeout(time: 16, unit: 'HOURS')
    disableConcurrentBuilds()
    // as we "checkout scm" as a stage, we do not need to do it here too
    skipDefaultCheckout(true)
  }

  stages {
    stage("Checkout") {
      steps {
        checkout scm
      }
    }

    stage("Build and Test") {
//      options {
//            // lock a shared resource in case two different EPICS builds try to run
//            lock(resource: ELOCK, inversePrecedence: true)
//      }
      stages {
        stage("Build") { 
          steps {
		    lock(resource: ELOCK, inversePrecedence: true) {
            echo "Branch: ${env.BRANCH_NAME}"
            echo "Build Number: ${env.BUILD_NUMBER}"
            script {
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
            echo ${env.WORKSPACE} ${env.ZIPNAME} ${params.EPICS_HOST_ARCH} ${env.CLEAN_BUILD}
            """
          }
		  }
        }

        stage("Test") { 
          steps {
            echo "Testing"
          }
        }
      }
    }

    stage("Deploy") {
          steps {
            echo "Deploying"
          }
    }
  }
}
