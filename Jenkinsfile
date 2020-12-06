pipeline {
  agent { label "master" }

  parameters {
    booleanParam(name: 'DEBUG', defaultValue: false, description: 'Build with debug symbols')
  }

  triggers {
    // enable weekly rebuilds
    cron('H H(6-9) * * 1')
  }

  options {
    disableConcurrentBuilds()
    parallelsAlwaysFailFast()
  }

  environment {
    DEBUG = "${params.DEBUG}"
    SKIP_CLEAN_CHECKOUT = "yes"
  }

  stages {

    stage('init') {
      // execute at master
      steps {
        echo "Libximc init"
      }
    } // stage

    stage('build') {
      matrix {
        // limit execution at agent with matching label
        agent {
          label "${env.BUILDOS}"
        }
        axes {
          axis {
            name 'BUILDOS'
            values 'debian64', 'debian32', 'suse64', 'suse32', 'win', 'osx'
          }
        }
        stages {

          stage('prebuild') {
            steps {
              echo "Pre"
              echo "Building at BUILDOS=${BUILDOS}"
            }
          } // stage

          stage('build-unix') {
            // Build for Linux and OSX agents
            when { expression { isUnix() } }
            steps {
              sh "./build-ci build"
              stash name: "result-${BUILDOS}", includes: "results-dist-*.tar"
            }
          } // stage

          stage('build-win') {
            // Build for Windows agents
            when { expression { !isUnix() } }
            steps {
              bat "build-ci.bat"
              stash name: "result-${BUILDOS}", includes: "results-dist-*.tar"
            }
          } // stage
        } // stages
        post {
          cleanup {
            // drop workspace for each matrix cell job
            deleteDir()
          }
        }
      } // matrix
    } // stage
    
    stage('docs') {
      // execute at master
      steps {
        // Just to be sure delete any local leftover files
        sh "rm -rf deps"
        sh "./build-ci docs"
        stash name: "result-docs", includes: "results-dist-*.tar"
      }
    } // stage


    stage('pack') {
      // execute on master
      steps {
        // Get all stashed archives
        unstash "result-debian64"
        unstash "result-debian32"
        unstash "result-suse64"
        unstash "result-suse32"
        unstash "result-win"
        unstash "result-osx"
        unstash "result-docs"
        sh "ls"
        sh "./build-ci dist"
        archiveArtifacts artifacts: "dist/ximc*.tar.gz"
      }
    } // stage

  } // stages

  post {
    failure {
      echo "Failure, sending emails..."
      //emailext body: '$DEFAULT_CONTENT',
      //         to: '$DEFAULT_RECIPIENTS',
      //         recipientProviders: [[$class: 'DevelopersRecipientProvider'],[$class: 'CulpritsRecipientProvider']],
      //         subject: '$DEFAULT_SUBJECT'
    }
    cleanup {
      // drop workspace for main job
      deleteDir()
    }
  }
}
