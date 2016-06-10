import groovy.json.JsonOutput

/* Send a notification to a #channel in Slack
*
* @param message string message to be sent
* @param channel channel where the message will be sent
*/
def slackNotificacao(message, channel) {
  // Create a webhook in slack and add the URL in slackURL
  def slackURL = 'https://hooks.slack.com/services/yourIncomingWeebHook'
  def payload = JsonOutput.toJson([text      : message,
    channel   : channel,
    username  : "jenkins"])
    sh "curl -X POST --data-urlencode \'payload=${payload}\' ${slackURL}"
}

node {
  /* Identify the tag version in a git repository and
   * build a package with this version 
   */
  stage 'Build'
  checkout([$class: 'GitSCM',
    branches: [[name: '*/master']],
    doGenerateSubmoduleConfigurations: false,
    extensions: [],
    submoduleCfg: [],
    userRemoteConfigs: [[
      credentialsId: 'xxxx-xsXAqx-xq!@K#xx',
      url: 'gitUrl' // Gitlab or Github services
    ]]
  ])
  // Get the tag version
  sh ('git describe --abbrev=0 --tags > TAG_VERSION')
  def TAG = readFile('TAG_VERSION')
  def TAG_VERSION = TAG.replaceAll("\\s","")

  // Run the job to build a version package based in TAG_VERSION
  build job:'job_build',
  parameters: [[$class: 'StringParameterValue',
  name: 'BUILD_VERSION',
  value: TAG_VERSION]]
}
