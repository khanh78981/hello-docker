pipeline {
  agent { label 'docker' }
  environment {
    REGISTRY = "${params.REGISTRY}"
    IMAGE    = "${env.REGISTRY}/hello:${env.GIT_COMMIT?.take(7) ?: env.BUILD_NUMBER}"
    APP_PORT = "32080"
    CONTAINER = "hello-web"
  }
  parameters {
    string(name: 'REGISTRY', defaultValue: '192.168.0.100:5000',
           description: 'Docker registry (vd: 192.168.0.100:5000)')
  }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Prepare app') {
      steps {
        sh '''
          cat > index.html <<HTML
          <html><body style="font-family:sans-serif">
          <h1>Hello from Jenkins #${BUILD_NUMBER}</h1>
          </body></html>
          HTML
        '''
      }
    }

    stage('Build image') { steps { sh 'docker build -t $IMAGE .' } }

    stage('Push image') { steps { sh 'docker push $IMAGE' } }  // *** thêm bước push ***

    stage('Run container') {
      steps {
        sh '''
          docker rm -f $CONTAINER >/dev/null 2>&1 || true
          docker run -d --name $CONTAINER -p 192.168.0.100:$APP_PORT:80 $IMAGE
          sleep 2
        '''
      }
    }

    stage('Smoke test') {
      steps {
        sh '''
          docker run --rm --network=container:$CONTAINER curlimages/curl:8.8.0 \
            -fsS --retry 5 --connect-timeout 2 http://127.0.0.1/ | head -n1
        '''
      }
    }
  }
  post { failure { sh 'docker logs $CONTAINER || true' } }
}
