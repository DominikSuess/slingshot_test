node {
  git '/tmp/repo'

  def maven = docker.image('maven:3.3.9-jdk-8'); // https://registry.hub.docker.com/_/maven/
  def slingImg = docker.image("apachesling/sling");

  stage('Mirror') {
    // First make sure the slave has this image. 
    // (If you could set your registry below to mirror Docker Hub,
    // this would be unnecessary as maven.inside would pull the image.)
    maven.pull()
    slingImg.pull()
  }
  
  // we're globaly locking this resource (avoid parallelization, as we need to run on a shared docker environment)
  // if we have distinct slaves we can lock the resource for the concrete environment and have parallel builds on multiple slaves
  lock('docker-and-scm-workspace') {
  
    // We are pushing to a private secure Docker registry in this demo.
    // 'docker-registry-login' is the username/password credentials ID as defined in Jenkins Credentials.
    // This is used to authenticate the Docker client to the registry.
    docker.withRegistry('https://localhost/', 'docker-registry-login') {
      def slingContainer
      try {
        stage('Build') {
          sshagent (credentials: ['github_ssh']) {
            checkout scm
            // in case of parallelization we might need to push the merged state to some jenkins-branch
            // for inter slave communication and not merge from master but from jenkinsbranch
            sh "git fetch --all"
            sh "git rebase origin/${env.CHANGE_TARGET}"
            maven.inside {
              sh "mvn clean package" 
            }
          }
        }
          
        stage('Testing') {
          parallel(Integration: {
            slingContainer = slingImg.run('-p 8090:8080')
            env.SLING_CONTAINER_ID = slingContainer.id
            maven.inside {
              echo 'test'
              // this should deploy on the container - port due to unkown reasons unreachable
              // sh "mvn sling:install -Dsling.url=http://localhost:8090/system/console"   
            }
          }, StaticAnalysis: { 
            echo 'Mocked static testing'
          })
        }
    
        // Milestone should be set - we only need to release in case there where no newer builds succeeding
        // but hitting https://issues.jenkins-ci.org/browse/JENKINS-38464
        stage('Release & Baseline') {
          echo "Release & Merge"
          sshagent (credentials: ['github_ssh']) {
            sh "git checkout ${env.CHANGE_TARGET}"
            sh "git merge --no-ff --log -m 'Merge pull request' ${env.BRANCH_NAME}"
            sh "git diff"
            sh "git commit"
            sh "git push --set-upstream origin ${env.CHANGE_TARGET}"
          }
          sh "docker commit ${env.SLING_CONTAINER_ID} apachesling/sling:latest"
          // make sure reference is really to the latest
          slingImg = docker.image("apachesling/sling")
          // push to private registry
          slingImg.push()
        }  
      } finally {
          slingContainer.stop()
      }  
    }
  }

  stage('Staging') {
    parallel(Endurance: {
           echo 'Mocking endurance Testing'
          }, UserAcceptance: { 
            input 'Confirm User Acceptance'
          })
  }
  
  stage('Deploy to production') {
    sh "echo 'deploy production'"
  }
}
