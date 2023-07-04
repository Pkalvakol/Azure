import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=00b62b12-21d1-43ab-8752-85671481ba87',
        'AZURE_TENANT_ID=025f89bd-eea6-4a20-b985-762459689deb']) {
    
    stage('init') {
      checkout scm
    }
  
    stage('building') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'DefaultResourceGroup-EUS'
      def webAppName = 'DefaultResourceGroup-EUS'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'azurework', passwordVariable: '7AL8Q~9IhyDPsmJ5iKRiA2KxKAR9J32v_5klIaPo', usernameVariable: '0957c43d-68c5-448b-a9cf-aee8efdb38ae')]) {
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
