#!groovy

def jdk = 'jdk8'

node("linux") {
  def mvntool = tool name: 'maven3.5', type: 'hudson.tasks.Maven$MavenInstallation'
  def mvntoolInvoker = tool name: 'maven3.5', type: 'hudson.tasks.Maven$MavenInstallation'
  def jdktool = tool name: "$jdk", type: 'hudson.model.JDK'
  def mvnName = 'maven3.5'
  def localRepo = "${env.JENKINS_HOME}/${env.EXECUTOR_NUMBER}" // ".repository" //
  def settingsName = 'oss-settings.xml'
  def mavenOpts = '-Xms1g -Xmx4g -Djava.awt.headless=true'

  // Environment
  List mvnEnv = ["PATH+MVN=${mvntool}/bin", "PATH+JDK=${jdktool}/bin", "JAVA_HOME=${jdktool}/", "MAVEN_HOME=${mvntool}"]
  mvnEnv.add("MAVEN_OPTS=$mavenOpts")

  stage("Checkout") {
    checkout scm
  }

  stage("Build") {
    withEnv(mvnEnv) {
      timeout(time: 90, unit: 'MINUTES') {
        // Run test phase / ignore test failures
        withMaven(
            maven: mvnName,
            jdk: "$jdk",
            publisherStrategy: 'EXPLICIT',
            globalMavenSettingsConfig: settingsName,
            //options: [invokerPublisher(disabled: false)],
            mavenOpts: mavenOpts,
            mavenLocalRepo: localRepo) {
          sh "mvn -V -B install -Dmaven.test.failure.ignore=true -e -DmavenHome=${mvntoolInvoker}"
        }
        // Report failures in the jenkins UI
        junit testResults: '**/target/surefire-reports/TEST-*.xml,**/target/failsafe-reports/TEST-*.xml'
        consoleParsers = [[parserName: 'JavaDoc'],
                          [parserName: 'JavaC']];
      }
    }
  }
}

// vim: et:ts=2:sw=2:ft=groovy
