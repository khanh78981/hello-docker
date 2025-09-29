pipeline {
  agent any
  environment {
    IMAGE = "lab/hello:${env.BUILD_NUMBER}"
    APP_PORT = "32080"
    CONTAINER = "hello-web"
  }
  triggers { pollSCM('H/1 * * * *') } // webhook sẽ là chính, cái này dự phòng
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Prepare app') {
      steps {
        sh '''
          cat > index.html <<HTML
          <html><body style="font-family:sans-serif">
          <h1>Hello from Jenkins build #${BUILD_NUMBER}</h1>
          </body></html>
          HTML
          ls -l
        '''
      }
    }
    stage('Build image') { steps { sh 'docker build -t $IMAGE .' } }
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
      steps { sh 'curl -fsS --retry 3 http://192.168.0.100:$APP_PORT/ | head -n1' }
    }
  }
  post { failure { sh 'docker logs $CONTAINER || true' } }
}
