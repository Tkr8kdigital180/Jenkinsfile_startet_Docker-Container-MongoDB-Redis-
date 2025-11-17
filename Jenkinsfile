pipeline {
  agent any

  environment {
    MONGO_NAME = 'ci-mongo'
    REDIS_NAME = 'ci-redis'
  }

  stages {
    stage('Docker: Pull Images') {
      steps {
        sh '''
          docker pull mongo:7
          docker pull redis:7
        '''
      }
    }

    stage('Docker: Run Containers') {
      steps {
        sh '''
          docker run -d --rm --name "$MONGO_NAME" -p 27017:27017 mongo:7
          docker run -d --rm --name "$REDIS_NAME" -p 6379:6379 redis:7
          echo "Laufende Container:"
          docker ps
        '''
      }
    }

    stage('Verify Services') {
      steps {
        sh '''
          echo "Prüfe Redis mit PING:"
          docker exec "$REDIS_NAME" redis-cli ping

          echo "Prüfe, ob Mongo-Prozess läuft:"
          docker exec "$MONGO_NAME" ps aux | grep mongod | grep -v grep
        '''
      }
    }
  }

  post {
    always {
      echo 'Räume Docker-Container auf...'
      sh '''
        docker stop "$REDIS_NAME" || true
        docker stop "$MONGO_NAME" || true
      '''
    }
    success {
      echo 'Docker-Check erfolgreich.'
    }
    failure {
      echo 'Docker-Check fehlgeschlagen – Logs prüfen!'
      sh '''
        echo "--- Redis Logs ---"
        docker logs "$REDIS_NAME" || true
        echo "--- Mongo Logs ---"
        docker logs "$MONGO_NAME" || true
      '''
    }
  }
}
