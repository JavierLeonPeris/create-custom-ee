pipeline {
  environment {
    ARTIFACTORY_CREDS = credentials('artifactory')
    REGISTRY_CREDS = credentials('automation-hub')
    AUTOMATION_HUB_TOKEN = credentials('aaphub-dev')
    AUTOMATION_HUB_URL = 'https://aaphub-dev.aib.pri/api/galaxy/'
    REGISTRY = 'aaphub-test.aib.pri'
    IMAGE = 'dbbe-a2.15-p3.9-rhel9.3'
  }
  agent { label 'docker' }
  stages {
    stage('Clone') {
      steps {
        git branch: 'master', url: 'ssh://git@gitstash.aib.pri/midap/dbbe-a2.15-p3.9-rhel9.3.git'
        script {
            env.GIT_COMMIT_EMAIL=sh(returnStdout: true, script: "git log --no-merges -s --format='%ae' -1")
            env.CI_VERSION = sh(returnStdout: true, script: "${WORKSPACE}/.ci/calculate-pipeline-build-number.sh")
            env.CURRENT_DATE=sh(returnStdout: true, script: "date '+%Y-%m-%d_%H-%M'")
            currentBuild.displayName = "${CI_VERSION}"
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'ansible-builder build --verbosity 3 --prune-images --build-arg ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_URL=${AUTOMATION_HUB_URL} --build-arg ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN=${AUTOMATION_HUB_TOKEN} --container-runtime docker -t ${REGISTRY}/${IMAGE}:${CI_VERSION} -t ${REGISTRY}/${IMAGE}:${CURRENT_DATE} -t ${REGISTRY}/${IMAGE}:latest'
      }
    }
    stage('Push image to Automation Hub') {
      steps {
        sh '''
          echo ${REGISTRY_CREDS_PSW} | docker login -u ${REGISTRY_CREDS_USR} --password-stdin ${REGISTRY}
          docker push ${REGISTRY}/${IMAGE}:${CI_VERSION}
          docker push ${REGISTRY}/${IMAGE}:${CURRENT_DATE}
          docker push ${REGISTRY}/${IMAGE}:latest
        '''
      }
    }
    stage('Remove unused Docker Image') {
      steps{
        sh '''
            docker rmi $REGISTRY/$IMAGE:$CI_VERSION
            docker rmi $REGISTRY/$IMAGE:$CURRENT_DATE
            docker rmi $REGISTRY/$IMAGE:latest
        '''
      }
    }
  }
  post {
    success {
      emailext (
        subject: "SUCCESSFUL: Job '${JOB_NAME} [${CI_VERSION}]'",
        mimeType: "text/html",
        body: """<p>SUCCESSFUL: Job '${JOB_NAME} [${CI_VERSION}]':</p>
        <p>Check console output at &QUOT;<a href='${BUILD_URL}'>${JOB_NAME} [${CI_VERSION}]</a>&QUOT;</p>""",
        to: "cloudengineering@aib.ie"
      )
    }
    failure {
      emailext (
        subject: "FAILED: Job '${JOB_NAME} [${CI_VERSION}]'",
        mimeType: "text/html",
        body: """<p>FAILED: Job '${JOB_NAME} [${CI_VERSION}]':</p>
        <p>Check console output at &QUOT;<a href='${BUILD_URL}'>${JOB_NAME} [${CI_VERSION}]</a>&QUOT;</p>""",
        to: "${GIT_COMMIT_EMAIL}"
      )
    }
  }
}
