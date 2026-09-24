pipeline {
    agent any

    environment {
        PORT = "4000"
        CLOUDINARY_API_KEY = "819224586478434"
        CLOUDINARY_API_SECRET = "yr_TbbT_JDG2Xfu4KcrrcykeJvA"
        CLOUDINARY_CLOUD_NAME = "dtgsdprcl"
        FRONTEND_URL = "http://localhost:5173"
        DB_URL = "mongodb://mongo_container_training:27017/ecommerce_db"
        JWT_SECRET_KEY = "fgjfgsudgfudnhfousidfnewuikfmlewf"
        JWT_EXPIRE = "7d"
        COOKIE_EXPIRE = "7"
        NODE_ENV = "development"
        ALERT_EMAIL = "your-email@example.com"   // change this
    }

    stages {
        stage('git clone') {
            steps {
                sh "echo ${env.Testvar}"
                echo 'Hello World'

                git branch: 'main', url: 'https://github.com/Minar-bracu/react-job-portal-jenkins-pipeline'

                sh 'ls -lah'
            }
        }

        stage("Docker file creation") {
            steps {
                dir("backend") {
                    writeFile file: 'Dockerfile', text: """
                    FROM node:20-slim as builder
                    RUN adduser --disabled-password bjit && chown -R bjit /app

                    USER bjit

                    WORKDIR /app

                    COPY package*.json ./

                    RUN npm install

                    FROM node:20-slim

                    WORKDIR /app
                    RUN adduser --disabled-password bjit && chown -R bjit /app

                    USER bjit

                    COPY --from=builder /app/node_modules ./node_modules

                    COPY . .


                    CMD ["node", "server.js"]

                    """
                }
            }
        }

        stage("Docker image build") {
            steps {
                dir("backend") {
                    writeFile file: 'docker-compose.yml', text: """
services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: backend_container_training
    restart: always
    ports:
      - "4000:4000"
    environment:
      - PORT=${env.PORT}
      - CLOUDINARY_API_KEY=${env.CLOUDINARY_API_KEY}
      - CLOUDINARY_API_SECRET=${env.CLOUDINARY_API_SECRET}
      - CLOUDINARY_CLOUD_NAME=${env.CLOUDINARY_CLOUD_NAME}
      - FRONTEND_URL=${env.FRONTEND_URL}
      - DB_URL=${env.DB_URL}
      - JWT_SECRET_KEY=${env.JWT_SECRET_KEY}
      - JWT_EXPIRE=${env.JWT_EXPIRE}
      - COOKIE_EXPIRE=${env.COOKIE_EXPIRE}
      - NODE_ENV=${env.NODE_ENV}
    networks:
      - project_3

networks:
  project_3:
    external: true

                    """
                }
            }
        }

        stage("Docker compose run") {
            steps {
                dir("backend") {
                    sh "docker compose down"
                    sh "docker compose up -d --build"
                }
            }
        }
    }

    post {
        always {
            echo "🔁 Pipeline finished with status: ${currentBuild.currentResult}"
            // Optional: clean the workspace after every run
            // cleanWs()
        }

        success {
            echo "✅ Deployment successful! I am from success block"

            emailext(
                to: "${env.ALERT_EMAIL}",
                subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """
                    <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> succeeded.</p>
                    <p>Details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }

        failure {
            echo "❌ Deployment failed! I am from failure block"

            emailext(
                to: "${env.ALERT_EMAIL}",
                subject: "Jenkins Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                mimeType: 'text/html',
                body: """
                    <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> failed.</p>
                    <p>Console: <a href="${env.BUILD_URL}console">${env.BUILD_URL}console</a></p>
                    <p>The full console log is attached.</p>
                """,
                attachLog: true,     // attaches build.log
                compressLog: false   // plain text, not .gz
            )
        }
    }
}
