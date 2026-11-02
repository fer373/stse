pipeline {
  agent any

  // Cek repo GitHub tiap ~2 menit; ada commit baru -> pipeline jalan otomatis
  triggers {
    pollSCM('H/2 * * * *')
  }

  stages {
    stage('Test') {
      agent {
        docker {
          image 'node:22-alpine'
          reuseNode true
        }
      }
      steps {
        // Koneksi VPS ke registry npm kadang tersendat (ETIMEDOUT), jadi unduhan diulang bila gagal.
        // Test sengaja tidak di-retry: test gagal harus langsung membuat build merah.
        retry(3) {
          sh 'npm ci --cache .npm'
        }
        sh 'npm run test:ci'
      }
      post {
        always {
          junit testResults: 'reports/junit.xml', allowEmptyResults: true
        }
      }
    }

    stage('Build Image') {
      steps {
        retry(3) {
          sh 'docker compose build'
        }
      }
    }

    stage('Deploy') {
      steps {
        sh 'docker compose up -d --wait'
      }
    }
  }

  post {
    success {
      echo 'Pipeline berhasil: cart-api berjalan di port 3001'
    }
    failure {
      echo 'Pipeline gagal: cek stage yang berwarna merah'
    }
  }
}
