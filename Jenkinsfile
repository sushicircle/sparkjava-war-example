  // git credentials
  def gitCredentials = "gitlogin"
  // git repo
  def gitrepo        = 'https://github.com/sushicircle/sparkjava-war-example.git'
  // default branch
  def branches       = 'origin/master'

// retrieve all branches from repo
def getBranches(gitrepo) {
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
    
    //deleteDir()
  
    stage('checkout') {
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
