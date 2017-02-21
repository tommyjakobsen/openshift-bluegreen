#!groovy
// This pipeline expects a parameter COLOR which will set the color of the newly deployed application.

node {
  // Blue/Green Deployment into Production
  // -------------------------------------
  def dest     = "example-green"
  def active   = ""
  def newcolor = "purple"

  if (getBinding().hasVariable("COLOR")) {
    newcolor = ${COLOR}
  }
  else {
    echo "No parameter for color."
  }
  sh "oc project bluegreen"

  stage('Determine Deployment Color') {
    sh "oc get route example -n bluegreen -o jsonpath='{ .spec.to.name }' > activesvc.txt"
    active = readFile('activesvc.txt').trim()
    if (active == "example-green") {
      dest = "example-blue"
    }
    echo "Active svc: " + active
    echo "Dest svc:   " + dest
  }

  stage('Build new version') {
    // Building in this case means simply changing the environment variable COLOR in the deployment configuration.
    // There is not Build Configuration since this is a straight up Docker Image deployment.
    echo "Building ${dest}"
    sh "oc set env dc ${dest} COLOR=${newcolor}"
  }

  stage('Deploy new Version') {
    echo "Deploying to ${dest}"

    openshiftDeploy depCfg: dest, namespace: 'bluegreen', verbose: 'false', waitTime: '', waitUnit: 'sec'
    openshiftVerifyDeployment depCfg: dest, namespace: 'bluegreen', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'true', waitTime: '', waitUnit: 'sec'
    openshiftVerifyService namespace: 'bluegreen', svcName: dest, verbose: 'false'
  }
  stage('Switch over to new Version') {
    input "Switch Production?"
    sh 'oc patch route example -n bluegreen -p \'{"spec":{"to":{"name":"' + dest + '"}}}\''
    sh 'oc get route example -n bluegreen > oc_out.txt'
    oc_out = readFile('oc_out.txt')
    echo "Current route configuration: " + oc_out
  }
}
