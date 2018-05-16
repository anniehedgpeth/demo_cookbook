// The cookbook and current branch that is being built
def branch = env.BRANCH_NAME
def cookbook = 'demo_cookbook'
currentBuild.displayName = "#${BUILD_NUMBER}; Branch: ${branch}"

// // OTHER (Unchanged)
// // **** Everything below should not need to change ****

// // the checkout directory for the cookbook; usually not changed
def cookbookDirectory = "cookbooks/${cookbook}"

// // Configuring the Stash Notifier in Jenkins to talk to BitBucket
def notifyStash(String sha) {
  // def sha1 = get_sha1(cookbookDirectory)
  step([
    $class: 'StashNotifier',
    commitSha1: sha, 
    considerUnstableAsSuccess: false,
    credentialsId: '5adcc81c-d389-4265-bfd6-1f5bb9a880ef', // Credential in Jenkins corresponds to the scm user (safe for source control)
    disableInprogressNotification: false,
    ignoreUnverifiedSSLPeer: true,
    includeBuildNumberInKey: false,
    prependParentProjectKey: false,
    projectKey: '',
    stashServerBaseUrl: 'https://git.kcura.com' // URL to BitBucket
  ])
}

// // To run the rake commands in the rakefile
def rake(command) {
  bat "chef exec rake ${command} CI=false"
}

// // Cookbook build and returning results to BitBucket
node() {
  stage('Validate Parameters') {
    echo "cookbook: ${cookbook}"
    echo "current branch: ${branch}"
    echo "checkout directory: ${cookbookDirectory}"
    try {
      // subscriptionId
      branch
      scmProps = checkout scm
    } catch (MissingPropertyException mpe) {
      echo "Pipeline parameters have been updated, please re-build"
      throw mpe;
    }
  }
  stage('Lint') {
    notifyStash(scmProps.GIT_COMMIT)
    try {
      rake('clean')
      rake('style')
    }
    currentBuild.result = 'SUCCESS'
    catch(err){
      currentBuild.result = 'FAILED'
      notifyStash(scmProps.GIT_COMMIT)
      throw err
    }
  }
  stage('Functional (Kitchen)') {
    try{
      rake('test')
      currentBuild.result = 'SUCCESS'
    }
    catch(err){
      currentBuild.result = 'FAILED'
      throw err
    }
    finally{
      notifyStash(scmProps.GIT_COMMIT)
    }
  }
}