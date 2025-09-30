pipeline {
  agent { label 'docker' }

  parameters {
    string(name: 'REGISTRY', defaultValue: '192.168.0.100:5000', description: 'Docker registry (vd: 192.168.0.100:5000)')
  }

  environment {
    DOCKER_HOST = 'tcp://docker-proxy:2375'
    REGISTRY   = "${params.REGISTRY}"
    APP_PORT   = "32080"
    CONTAINER  = "hello-web"
  }

  options { timestamps() }

  stages {

    stage('Sanity Docker via proxy') {
      steps {
        sh '''
          echo "DOCKER_HOST=$DOCKER_HOST"
          docker version
          docker info | sed -n "1,20p"
        '''
      }
    }

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Set vars') {
      steps {
        script {
          env.GIT_SHORT = sh(script: 'git rev-parse --short=7 HEAD', returnStdout: true).trim()
          env.IMAGE = "${params.REGISTRY}/hello:${GIT_SHORT}"
          echo "REGISTRY=${params.REGISTRY}"
          echo "IMAGE=${env.IMAGE}"
        }
      }
    }

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

    stage('Build image')   { steps { sh 'docker build -t "$IMAGE" .' } }
    stage('Push image')    { steps { sh 'docker push "$IMAGE"'       } }

    stage('Run container') {
      steps {
        sh '''
          docker rm -f "$CONTAINER" >/dev/null 2>&1 || true
          docker run -d --name "$CONTAINER" --restart unless-stopped \
            -p 192.168.0.100:"$APP_PORT":80 "$IMAGE"
          sleep 2
        '''
      }
    }

    stage('Smoke test') {
      steps {
        sh '''
          docker run --rm --network=container:"$CONTAINER" curlimages/curl:8.10.1 \
            -fsSI http://127.0.0.1/ | head -n1 | grep -q "200"
          echo "Smoke OK"
        '''
      }
    }
  }

  post {
    always  { sh 'docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}" || true' }
    failure { sh 'docker logs "$CONTAINER" || true' }
  }
}
