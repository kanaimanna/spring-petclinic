pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3.9'
    }

    environment {
        APP_NAME = 'petclinic'
        APP_PORT = '8081'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kanaimanna/spring-petclinic.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Run Application') {
            steps {
                sh '''
                    # Stop any previous instance running on APP_PORT
                    fuser -k ${APP_PORT}/tcp || true
                    sleep 2

                    # Prevent Jenkins from killing this process once the build step ends.
                    # Both variables must be unset - newer Jenkins uses JENKINS_NODE_COOKIE,
                    # older versions use BUILD_ID.
                    export BUILD_ID=dontKillMe
                    export JENKINS_NODE_COOKIE=dontKillMe

                    # setsid fully detaches the process into its own session, so it survives
                    # even if Jenkins tries to kill the whole process group.
                    # (no 'disown' here - it's a bash builtin, not available in Jenkins' /bin/sh)
                    setsid nohup java -jar target/*.jar --server.port=${APP_PORT} > app.log 2>&1 < /dev/null &

                    sleep 15

                    echo "Checking if the application actually started..."
                    if ss -tulpn | grep -q ":${APP_PORT}"; then
                        echo "SUCCESS: Application is listening on port ${APP_PORT}"
                    else
                        echo "WARNING: Nothing is listening on port ${APP_PORT} yet. Log so far:"
                    fi
                    tail -n 30 app.log
                '''
            }
        }
    }

    post {
        success {
            echo "Build #${env.BUILD_NUMBER} succeeded. ${APP_NAME} is running on port ${APP_PORT}."
        }
        failure {
            echo "Build #${env.BUILD_NUMBER} failed. Check the console output above for details."
        }
    }
}
