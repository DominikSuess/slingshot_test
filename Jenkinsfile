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

  // We are pushing to a private secure Docker registry in this demo.
  // 'docker-registry-login' is the username/password credentials ID as defined in Jenkins Credentials.
  // This is used to authenticate the Docker client to the registry.
  docker.withRegistry('https://localhost/', 'docker-registry-login') {

    stage('Build') {
      sshagent (credentials: ['github_ssh']) {
        checkout scm
        sh "git merge origin/${env.CHANGE_TARGET}"
        sh "git push origin HEAD:${env.CHANGE_TARGET}"
      
        maven.inside {
 
          echo "My branch is: ${env.BRANCH_NAME}"
          echo "My target branch is: ${env.CHANGE_TARGET}"
          //  "mvn clean package" 

        }
      }
    }

    stage('Run Docker image') {
        slingImg.inside {
            sh "echo 'test'"
        }
    }


    stage('Promote Image') {
        slingImg.push();
    }
  }
}
