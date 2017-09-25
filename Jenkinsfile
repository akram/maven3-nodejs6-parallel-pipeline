try {
  timeout(time: 20, unit: 'MINUTES') {
      def appName="rma"
      def project=""

      node {
        stage("Initialize") {
          project = env.PROJECT_NAME
          stage("Checkout") {
          //checkout scm
          git url: 'https://github.com/akram/angular-wildfly-basic-seed.git', branch: 'master'
          stash name:"frontend", includes:"frontend/"
	  stash name:"backend", includes:"backend/"
          }
        }
      }
      parallel ( 
        "backend": {
          node('maven35') {
            stage("Compile backend") {
              unstash name:"backend"
              sh "mvn clean package -Popenshift"
              stash name:"war", includes:"target/ROOT.war"
            }
          }
        },
        "frontend": {
          node('nodejs6') {
           stage("Compile frontend") {
  	     unstash name:"backend"
  	     sh "npm install ; ng build"
           }
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

