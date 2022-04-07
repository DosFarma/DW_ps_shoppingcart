pipeline { agent { kubernetes { inheritFrom 'php73' } }
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
        container('php') {
          sh 'apt-get -y update && apt-get install -y jq libicu-dev zlib1g  zlib1g-dev libzip-dev libpng-dev libjpeg-dev libwebp-dev libcurl4-openssl-dev libxml2 libxml2-dev'
          sh 'docker-php-ext-configure intl'
          sh 'docker-php-ext-install intl zip gd curl soap'
          sh 'mkdir -p ${BUILD_ROOT}'
          sh 'mkdir -p ${WEB_ROOT}'
          sh 'mkdir -p ${TO_SHARED_ROOT}'
          sh 'mkdir -p ${META_INF_ROOT}'
          sh 'curl http://getcomposer.org/installer > composer-setup.php'
          sh '''php -r "if (hash_file(\'sha384\', \'composer-setup.php\') === \'906a84df04cea2aa72f40b5f787e49f22d4c2f19492ac310e8cba5b96ac8b64115ac402c8cd292b8a03482574915d1a8\') { echo \'Installer verified\'; } else { echo \'Installer corrupt\'; unlink(\'composer-setup.php\'); } echo PHP_EOL;" '''
          sh 'php composer-setup.php'
          sh 'rm composer-setup.php'
          withCredentials([usernamePassword(credentialsId: 'satis-jenkins', passwordVariable: 'SATIS_JENKINS_PASSWORD', usernameVariable: 'SATIS_JENKINS_USERNAME')]) {
            sh 'php composer.phar config --global http-basic.jenkins.dosfarma.com $SATIS_JENKINS_USERNAME $SATIS_JENKINS_PASSWORD'
          }
        }
      }
    }
    stage('Prepare (composer all deps)') {
      steps {
        container('php') {
          sh 'php composer.phar install --no-ansi --no-interaction --dev'
        }
      }
    }
    stage('Tests') {
      steps {
        sh 'sleep 1s'
      }
    }
    stage('Prepare (composer without dev)') {
      steps {
        container('php') {
          sh 'php composer.phar install --no-dev --no-ansi --no-interaction --no-dev'
        }
      }
    }
  }
  post {
    always {
      emailext attachLog: true, body: """${currentBuild.currentResult}: Job: ${env.JOB_NAME} build: ${env.BUILD_NUMBER}

Recibes este correo bien porque has realizado el push/petición de integración o tienes cambios en el changeset procesado.
El log de la tarea ejecutada en el archivo anexo. """, compressLog: true, recipientProviders: [buildUser(), developers()], subject: "[Jenkins][${currentBuild.currentResult}]: Job: ${env.JOB_NAME} build: ${env.BUILD_NUMBER}", to: 'informatica@dosfarma.com, fj.rubio@dosfarma.com'
      office365ConnectorSend webhookUrl: "${env.OFFICE365_CONNECTOR_URL}",
         message: "${env.JOB_NAME} build: ${env.BUILD_NUMBER}",
         status: "${currentBuild.currentResult}"
    }
  }
}
