pipeline {
  // agent { kubernetes { inheritFrom 'php73' } }
  agent { label 'docker-jnlp-php' }

  environment {
    GIT_SHORT_ID = sh(returnStdout: true, script: 'git describe --always').trim()
    GIT_NAME = sh(returnStdout: true, script: 'git --no-pager show -s --pretty="format:%an"').trim()
    GIT_EMAIL = sh(returnStdout: true, script: 'git --no-pager show -s --pretty="format:%ae"').trim()
    GIT_COMMIT_ID = sh(returnStdout: true, script: 'git --no-pager show -s --pretty="format:%H"').trim()
    ARTIFACT_NAME = "${JOB_BASE_NAME}_${BUILD_ID}".trim()
    BUILD_ROOT = "build/${ARTIFACT_NAME}".trim()
    WEB_ROOT = "${BUILD_ROOT}/webroot".trim()
    TO_SHARED_ROOT = "${BUILD_ROOT}/ToShared".trim()
    META_INF_ROOT = "${BUILD_ROOT}/META-INF".trim()
    MANIFEST_FILE = "${META_INF_ROOT}/MANIFEST.MF".trim()
    ARTIFACT_FILENAME = "${BUILD_ROOT}.tgz".trim()
  }
  stages {
    stage('Prepare') {
      steps {
        // sh 'apt-get -y update && apt-get install -y php php-intl php-zip php-gd php-curl php-soap php-xml php-xmlwriter jq zlib1g libxml2 curl'
        sh 'mkdir -p ${BUILD_ROOT}'
        sh 'mkdir -p ${WEB_ROOT}'
        sh 'mkdir -p ${TO_SHARED_ROOT}'
        sh 'mkdir -p ${META_INF_ROOT}'
        // sh 'curl https://composer.github.io/installer.sha384sum > installer.sha384sum'
        // sh 'curl https://getcomposer.org/installer > composer-setup.php'
        // sh 'sha384sum -c installer.sha384sum'
        // sh 'php composer-setup.php --version=2.2.9'
        // sh 'rm composer-setup.php installer.sha384sum'
        withCredentials([usernamePassword(credentialsId: 'satis-jenkins', passwordVariable: 'SATIS_JENKINS_PASSWORD', usernameVariable: 'SATIS_JENKINS_USERNAME')]) {
          sh 'php /home/jenkins/composer.phar config --global http-basic.jenkins.dosfarma.com $SATIS_JENKINS_USERNAME $SATIS_JENKINS_PASSWORD'
        }
      }
    }
    stage('Prepare (composer all deps)') {
      steps {
        sh 'php /home/jenkins/composer.phar install --no-ansi --no-interaction --dev'
      }
    }
    stage('Tests') {
      steps {
        sh 'sleep 1s'
      }
    }
    stage('Prepare (composer without dev)') {
      steps {
        sh 'php /home/jenkins/composer.phar install --no-dev --no-ansi --no-interaction --no-dev'
      }
    }
  }
  post {
    always {
      emailext attachLog: true, body: """${currentBuild.currentResult}: Job: ${env.JOB_NAME} build: ${env.BUILD_NUMBER}

Recibes este correo bien porque has realizado el push/petición de integración o tienes cambios en el changeset procesado.
El log de la tarea ejecutada en el archivo anexo. """, compressLog: true, recipientProviders: [buildUser(), developers()], subject: "[Jenkins][${currentBuild.currentResult}]: Job: ${env.JOB_NAME} build: ${env.BUILD_NUMBER}", to: 'informatica@dosfarma.com, fj.rubio@dosfarma.com'
      withCredentials([string(credentialsId: 'office365-connector-url', variable: 'OFFICE365_URL')]) {
        office365ConnectorSend webhookUrl: env.OFFICE365_URL,
          message: "${env.JOB_NAME} build: ${env.BUILD_NUMBER}",
          status: "${currentBuild.currentResult}"
      }
    }
  }
}
