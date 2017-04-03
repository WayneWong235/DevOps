//test
def notifySlack(String buildStatus = 'STARTED') {
    // Build status of null means success.
    buildStatus = buildStatus ?: 'SUCCESS'

    def color

    if (buildStatus == 'STARTED') {
        color = '#D4DADF'
    } else if (buildStatus == 'SUCCESS') {
        color = '#BDFFC3'
    } else if (buildStatus == 'UNSTABLE') {
        color = '#FFFE89'
    } else {
        color = '#FF9FA1'
    }

    def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"

    slackSend(color: color, message: msg)
}
node {
 def mvnHome = tool 'M3'
 //stage('Configure') {
 //   env.PATH = "${tool 'M3'}/bin:${env.PATH}"
 // }
 stage('Checkout') {
    retry(3) {
     git credentialsId: 'ce6cf836-70c5-48fc-9bbb-2b8ab0c54775', url: 'git@git.kms-technology.com:coe/analytics-center.git'
    }
  }

  stage('Build') {
    //input message: 'Continue to Build', ok: 'Build'
    try {
        notifySlack()
        // Existing build steps.
        //waitUntil {
        //  try {
                sh "${mvnHome}/bin/mvn -f kac-maven-plugin/pom.xml clean install"
                nodejs(nodeJSInstallationName: 'default') {
                    sh "${mvnHome}/bin/mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=true -Pprod"
                    retry(3) {
                        sh "${mvnHome}/bin/mvn sonar:sonar -Psonar"
                    }
                }
        //  } catch(error) {
        //    input "Retry the job ?"
        //    false
        //  }
        //}
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        notifySlack(currentBuild.result)
    }
  }
  
  stage('Test') {
    junit '**/surefire-reports/TEST-*.xml'
    //step([$class: 'LastChangesPublisher', format: 'SIDE', matchWordsThreshold: '0.25', matching: 'NONE', matchingMaxComparisons: '1000', showFiles: true, synchronisedScroll: true])
    step([$class: 'JacocoPublisher', maximumBranchCoverage: '70', maximumClassCoverage: '100', maximumComplexityCoverage: '70', maximumInstructionCoverage: '70', maximumLineCoverage: '70', maximumMethodCoverage: '70'])
  }

}
