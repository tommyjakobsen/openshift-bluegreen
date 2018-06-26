import java.text.SimpleDateFormat;

// This pipeline expects a paramater newcolor which will set the newcolor of the newly deployed application.

node {
  // Blue/Green Deployment into Production
  // -------------------------------------
  //def rollback = ""
  def project  = ""
  def dest     = "green"
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
        newcolor="navy"
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
        newcolor="yellow"
        break
    case 9:
        newcolor="red"
        break
    case 0:
        newcolor="blue"
        break
  }

  stage('Determine Deployment color') {
    // Determine current project
    sh "oc get project|grep -v NAME|awk '{print \$1}' >project.txt"
    project = readFile('project.txt').trim()
    sh "oc get route green -n ${project} -o jsonpath='{ .spec.to.name }' > activesvc.txt"

    // Determine currently active Service
    active = readFile('activesvc.txt').trim()
    if (active == "green") {
      dest = "green"
     
    }
    echo "Active svc: " + active
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
    input "Switch "+newcolor+" version into Production?"
    sh 'oc patch route production -p \'{"spec":{"to":{"name":"' + dest + '"}}}\''
    sh 'oc patch route devops -p \'{"spec":{"to":{"name":"' + active + '"}}}\''
    sh 'oc get route production > oc_out.txt'
    sh 'oc get route devops > oc_out2.txt'
    oc_out = readFile('oc_out.txt')
    oc_out2 = readFile('oc_out2.txt')
    echo "Current route configuration production: " + oc_out
     echo "Current route configuration devops: " + oc_out
    
   
  }
  stage('Keep version?') {
  def userInput
try {
    userInput = input(
        id: 'Proceed1', message: 'Was this successful?', parameters: [
        [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']
        ])
} catch(err) { // input false
    def user = err.getCauses()[0].getUser()
    userInput = false
    echo "Aborted...."
}

node {
    if (userInput == true) {
        // do something
        echo "Deployment was successful"
    } else {
        // do something else
        echo "Rollback initiated"
       sh 'oc patch route production -p \'{"spec":{"to":{"name":"' + active + '"}}}\''
            sh 'oc patch route devops -p \'{"spec":{"to":{"name":"' + dest + '"}}}\''
            sh 'oc get route devops > oc_out.txt'
            sh 'oc get route production > oc_out2.txt'
            oc_out = readFile('oc_out.txt')
            oc_out2 = readFile('oc_out2.txt')
            echo "Current route configuration Production: " + oc_out2
            echo "Current route configuration devops: " + oc_out
        currentBuild.result = 'FAILURE'
    } 
}
      
}
   
  
  
  
    
    
  // stage('Hybrid Deploy') {
   // input "Dep. hybrid to " + hybriddest + "?"
   // echo "Deploying to :" +hybriddest + " with token:" + hybridtoken+"....."
    //echo "oc login " +hybriddest+" --token="+hybridtoken
     //sh "oc login ${hybriddest} --token=${hybridtoken} --insecure-skip-tls-verify=true"
     //sh "oc new-app https://github.com/tommyjakobsen/simple-php --name=public-app"
     //sh "oc set triggers dc/public-app --remove-all"
     //sh "oc expose svc/public-app --name=public"
     //sh "oc logout"

   
  //}
 
}
