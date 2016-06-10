import groovy.json.JsonOutput

/* Send a notification to a #channel in Slack
*
* @param message string message to be sent
* @param channel channel where the message will be sent
*/
def slackNotify(message, channel) {
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
  
  /*
   * Deploy in development enviroment
   */
  stage 'Development'
  resultDeployOk = false
  resultDeployOk = build job:'projectName-developmentDeploy',
  parameters: [[$class: 'StringParameterValue',
  name: 'TARGET_VERSION',
  value: TAG_VERSION]],
  propagate: false
  while (!resultDeployOk) {
    slackNotify("Deploy in development of `projectName v$TAG_VERSION` *failed*. <@slackUser> check this", 'delivery')

    input message: 'Development Deploy failed, start again?',
    parameters: [[$class: 'TextParameterDefinition',
    defaultValue: '',
    description: '']]

    resultadoDeployOk = build job:'projectName-developmentDeploy',
    parameters: [[$class: 'StringParameterValue',
    name: 'TARGET_VERSION',
    value: TAG_VERSION]],
    propagate: false
  }
  slackNotify("Deploy in *development* of `projectName v$TAG_VERSION` succeed. <@slackUser>", 'delivery')

  input message: "Approve the development version $TAG_VERSION?",
  parameters: [[$class: 'TextParameterDefinition',
  defaultValue: '',
  description: '',
  name: 'Observation']],
  //submitter: 'jenkinsRole'
}
