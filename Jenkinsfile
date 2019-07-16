node {
  stage('Checkout') {
    checkout scm
  }

  stage('Verify') {
    withCredentials([
        file(credentialsId: 'keybase-envfile',
             usernameVariable: 'KEYBASE_ENV_FILE')]) {
      def scmUrl = scm.getUserRemoteConfigs()[0].getUrl()
      sh '''
        docker run -it --env-file=${KEYBASE_ENV_FILE} \
          -e KEYBASE_TRUSTED_USERS=darrell-cox-training \
          -e GIT_USER_EMAIL="darrellcox19@googlemail.com" \
          -e GIT_REPO="${scmUrl}" \
          -e GIT_REVISIONS_TO_VERIFY=1 \
          controlplane/keybase:latest
      '''
  }

  stage('Build') {
    withCredentials([
        usernamePassword(credentialsId: 'docker-credentials',
                         usernameVariable: 'USERNAME',
                         passwordVariable: 'PASSWORD')]) {
      sh 'docker image build -t ${USERNAME}/demo-api:latest .'
    }
  }

  stage('Push') {
    withCredentials([
        usernamePassword(credentialsId: 'docker-credentials',
                         usernameVariable: 'USERNAME',
                         passwordVariable: 'PASSWORD')]) {
      sh 'docker login -p "${PASSWORD}" -u "${USERNAME}"'
      sh 'docker image push ${USERNAME}/demo-api:latest'
    }
  }

  stage('Deploy') {
    withCredentials([
        file(credentialsId: 'kube-config',
             variable: 'KUBECONFIG')]) {
      sh 'kubectl apply -f deployment.yaml'
    }
  }

  stage('Scan') {
    withCredentials([
	string(credentialsId: 'microscanner-token',
               variable: 'MICROSCANNER_TOKEN'),
        usernamePassword(credentialsId: 'docker-credentials',
                         usernameVariable: 'USERNAME',
                         passwordVariable: 'PASSWORD')]) {
      sh 'wget https://github.com/lukebond/microscanner-wrapper/raw/master/scan.sh -O /usr/local/bin/scan.sh && chmod +x /usr/local/bin/scan.sh'
      sh '/usr/local/bin/scan.sh ${USERNAME}/demo-api:latest'
    }
  }
}
