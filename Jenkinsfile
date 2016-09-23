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
      // Spin up a Maven container to build the petclinic app from source.
      // First set up a shared Maven repo so we don't need to download all dependencies on every build.
      maven.inside {
        checkout scm
        maven 'clean package'
        sh "mvn --version"
        // The app .war and Dockerfile are now available in the workspace. See below.
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
