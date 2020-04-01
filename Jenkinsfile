openshift.withCluster() {
  env.NAMESPACE = openshift.project()
  env.POM_FILE = env.BUILD_CONTEXT_DIR ? "${env.BUILD_CONTEXT_DIR}/pom.xml" : "pom.xml"
  env.APP_NAME = "spring-rest"
  echo "Starting Pipeline for ${APP_NAME}..."
}


pipeline {
  agent {
    label 'jenkins-slave-mvn' 
  }

  stages {

    stage('Build'){
      sh " mvn clean install -DskipTests=true -f ${POM_FILE}"
    }
  }

    stage('Unit Tests'){
      sh "mvn test -f ${POM_FILE}"
    }

    stage('Bake'){
      sh """
        rm -rf oc-build && mkdir -p oc-build/deployments
        for t in \$(echo "jar;war;ear" | tr ";" "\\n"); do
          cp -rfv ./target/*.\$t oc-build/deployments/ 2> /dev/null || echo "No \$t files"
        done
        oc start-build ${env.APP_NAME} --from-dir=oc-build --wait=true --follow=true || exit 1 
      """
    }

}


//  stage("Verify Deployment to ${env.STAGE1}") {
//
//    openshiftVerifyDeployment(deploymentConfig: "${env.APP_NAME}", namespace: "${STAGE1}", verifyReplicaCount: true)
//
//    input "Promote Application to Stage?"
//  }
//
//  stage("Promote To ${env.STAGE2}") {
//    sh """
//    ${env.OC_CMD} tag ${env.STAGE1}/${env.APP_NAME}:latest ${env.STAGE2}/${env.APP_NAME}:latest
//    """
//  }

