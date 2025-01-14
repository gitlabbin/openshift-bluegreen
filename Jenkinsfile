import java.text.SimpleDateFormat;

// This pipeline expects a paramater newcolor which will set the newcolor of the newly deployed application.

def getProject() {
    def projects = sh(script: "oc get project|grep -v NAME|awk '{print \$1}'", returnStdout: true)
    //sh "oc get route example -n ${project} -o jsonpath='{ .spec.to.name }' > activesvc.txt"
    def prjList = projects.readLines()
    echo "PRJ " + prjList[0]
    return prjList[0]
}

node {
  // Blue/Green Deployment into Production
  // -------------------------------------
  def project  = ""
  def dest     = "example-green"
  def active   = ""
  def newcolor = ""

  switch (BUILD_NUMBER.toInteger() % 10) {
    case 1:
        newcolor="red"
        break
    case 2:
        newcolor="maroon"
        break
    case 3:
        newcolor="yellow"
        break
    case 4:
        newcolor="magenta"
        break
    case 5:
        newcolor="purple"
        break
    case 6:
        newcolor="orange"
        break
    case 7:
        newcolor="cyan"
        break
    case 8:
        newcolor="navy"
        break
    case 9:
        newcolor="green"
        break
    case 0:
        newcolor="blue"
        break
  }

  stage('Determine Deployment color') {
    // Determine current project
    project = getProject()

    active = sh(script: "set +x; oc get route example -n ${project} -o jsonpath='{ .spec.to.name }'", returnStdout: true)


    // Determine currently active Service
    //active = readFile('activesvc.txt').trim()
    echo "Active svc: " + active
    if (active == "example-green") {
      dest = "example-blue"
    }
    echo "Dest svc:   " + dest
    echo "New color:  " + newcolor
  }

  stage('Build new version') {
    // Building in this case means simply changing the environment
    // variable newcolor in the deployment configuration.
    // There is not Build Configuration since this is a straight
    // up Docker Image deployment.
    echo "Building ${dest}"
    sh "oc set env dc ${dest} COLOR=${newcolor}"
  }

  stage('Deploy new Version') {
    echo "Deploying to ${dest}"

    openshiftDeploy depCfg: dest, namespace: project, verbose: 'false', waitTime: '', waitUnit: 'sec'
    openshiftVerifyDeployment depCfg: dest, namespace: project, replicaCount: '1', verbose: 'false', verifyReplicaCount: 'true', waitTime: '', waitUnit: 'sec'
    openshiftVerifyService namespace: project, svcName: dest, verbose: 'false'
  }
  stage('Switch over to new Version') {
    input "Switch Production?"
    sh 'oc patch route example -p \'{"spec":{"to":{"name":"' + dest + '"}}}\''
    sh 'oc get route example > oc_out.txt'
    oc_out = readFile('oc_out.txt')
    echo "Current route configuration: " + oc_out
  }
}
