  // git credentials
  def gitCredentials = "gitlogin"
  // git repo
  def gitrepo        = 'https://github.com/sushicircle/sparkjava-war-example.git'
  // default branch
  def branches       = 'origin/master'

// retrieve all branches from repo
def getBranches(gitrepo) {

    ansiColor('xterm') {
      echo "\033[31mused git repo: ${gitrepo}"
    }  
  
    def command     = "git ls-remote -h " + gitrepo
    def proc        = command.execute()

    // default branches
    def branches    = "origin/develop\norigin/release"

    proc.waitFor()

    if ( proc.exitValue() != 0 ) {
       println "Error, ${proc.err.text}"
    } else {
        def gitbranches = proc.in.text.readLines().collect {
            it.replaceAll(/[a-z0-9]*\trefs\/heads\//, '')
        }
        branches = gitbranches.join("\n")
    }
    return branches
}


node("master") {
  echo "master node"
  
  sshagent([gitCredentials]) {
    properties([
      parameters([
        text(name: 'first', defaultValue: '', description: 'anything'),
        choice(name: 'build_branch', choices: getBranches(gitrepo), description: 'source branceh')
      ])
    ])
  }
  try {
    notifyBuild('START')
    try {
      branch = params.build_branch
    } catch (e) {
      // do nothing, default to develop
    }
    
    ansiColor('xterm') {
      echo "\033[31mbranch: ${branch}"
    }  
    //deleteDir()
  
    stage('checkout repo') {
      sshagent([gitCredentials]) {
        checkout([
          $class: 'GitSCM',
          branches: [[
            name: branch
          ]],
          userRemoteConfigs: [[
            credentialsId: gitCredentials,
            url: gitrepo
          ]]
        ])
      }
    }
    stage('copy files') {
      sshagent([gitCredentials]) {
      checkout([
        $class: 'GitSCM',
        branches: [[
          name: branch
        ]],
        doGenerateSubmoduleConfigurations: false,
        extensions: [[
          $class: 'RelativeTargetDirectory',
          relativeTargetDir: 'dir/injenkins'
        ]],
        submoduleCfg: [],
        userRemoteConfigs: [[
          credentialsId: gitCredentials,
          url: gitrepo
        ]]
      ])
      }
      echo "workspace we copy from: ${WORKSPACE}"
      sh 'yes | sudo cp -vr ~/workspace/git\\ jenkinsfile\\ pipeline/dir/injenkins /home/milan/Documents/new'
      sh 'sudo chown milan:milan /home/milan/Documents/new/injenkins'
      
      //TODO copy to cloud server over ssh
      //def targetServer = 'prod'
      //def tomcat = 'tomcat'
      //sshPublisher(publishers: [sshPublisherDesc(configName: targetServer, transfers: [sshTransfer(excludes: '', execCommand: 'chown -R user:user /home/to/app/' + tomcat + '/webapps/war/WEB-INF/resources/config/* && ls -ltr /home/to/app/' + tomcat + '/webapps/war/WEB-INF/resources/config/*', execTimeout: 300000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'app/' + tomcat + '/webapps/war/WEB-INF/resources/config', remoteDirectorySDF: false, removePrefix: 'War/' + target_env + '/config', sourceFiles: 'War/' + target_env + '/config/*')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
        }
    } catch (e) {
        // If there was an exception thrown, the build failed
        currentBuild.result = "FAILED"
        throw e
    } finally {
        // Success or failure, always send notifications
        notifyBuild(currentBuild.result)
    }
}


  } catch (e) {
      // fail
      currentBuild.result = "FAILED"
      throw e
  } finally {
      // succes or fail, send notifications
      notifyBuild(currentBuild.result)
  }
  
  stage('check') {
  //TODO
    echo "check server config"
  }

}

def notifyBuild(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus = buildStatus ?: 'SUCCESS'

    // Default values
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    // Override default values based on build status
    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
    } else if (buildStatus == 'SUCCESS') {
        color = 'GREEN'
    } else {
        color = 'RED'
    }
}
