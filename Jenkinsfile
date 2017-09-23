try {
  timeout(time: 20, unit: 'MINUTES') {
      def appName="rma"
      def project=""

      node {
        stage("Initialize") {
          project = env.PROJECT_NAME
          stage("Checkout") {
          checkout scm
          stash name:"src", includes:"**"
          }
        }
      }
      parallel ( 
        "backend": {
          node('maven35') {
            stage("Compile backend")
            unstash name:"src"
            sh "mvn clean package -Popenshift"
            stash name:"war", includes:"target/ROOT.war"
          }
        },
        "frontend": {
          node('nodejs6') {
           stage("Compile frontend")
  	   unstash name:"src"
  	   sh "npm install ; ng build"
  	  }
  	}
      )

      node {
            stage("Build Image") {
              unstash name:"war"
              sh "oc start-build ${appName}-docker --from-file=target/ROOT.war -n ${project}"
              openshiftVerifyBuild bldCfg: "${appName}-docker", namespace: project, waitTime: '20', waitUnit: 'min'
            }
            stage("Deploy") {
              openshiftDeploy deploymentConfig: appName, namespace: project
            }
          }
  }
} catch (err) {
   echo "in catch block"
   echo "Caught: ${err}"
   currentBuild.result = 'FAILURE'
   throw err
}

