import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=126115f9-75e7-4789-9da4-ed851551fcd6',
        'AZURE_TENANT_ID=e4f8a9c5-0fb7-4f4d-b13a-a4c2abae4bf7']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'demoproj5'
      def webAppName = 'jenkinsdemo'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'f07797e6-238f-45ba-b9e9-68c529ba3788', passwordVariable: 'H3C8Q~n3EZJGj6OUY1UPIv8C4grwC~OWtVP.cdjP', usernameVariable: 'c2f95825-00de-4569-83ce-060d20505ccb')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
