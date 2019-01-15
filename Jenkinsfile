  // git credentials
  def gitCredentials = "gitlogin"
  // git repo
  def gitrepo        = 'https://github.com/sushicircle/sparkjava-war-example.git'
  // default branch
  def branches       = 'origin/master'

// retrieve branches
def get Branches(gitrepo) {
  def command   = "git ls-remote -h" + gitrepo
  def proc      = command.execute()
  
  proc.waitFor()
  
  if(proc.exitValue() != 0 {
    println "Error, ${proc.err.txt}"
  } else {
    def gitbranches = proc.in.text.readLines().collect {
      it.replaceAll(/[a-z0-9]*\trefs\/heads\//, '')
    }
    branches = gitbranches.join("\n")
  }
  println branches
  return branches
}

node("master) {
  echo "master node"
  
  stage('checkout') {
    sshagent([gitCredentials]) {
      checkout([
        $class: 'GITSCM',
        branches: [[
          name: branch
        ]],
        userRemoteConfigs: {{
          credentialsId: gitCredentials,
          url: gitrepo
        ]]
      ])
    }
  }
  
  stage('check') {
  //TODO
    echo "check server config"
  }

}