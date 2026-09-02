```groovy
pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3.9'
    }

    environment {
        APP_NAME = 'petclinic'
        TOMCAT_HOME = '/opt/tomcat'
        TOMCAT_PORT = '8082'
        WAR_NAME = 'petclinic.war'
    }

    stages {

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

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                    echo "Deploying ${APP_NAME} to Tomcat..."

                    # Remove previous deployment
                    sudo rm -rf ${TOMCAT_HOME}/webapps/${APP_NAME}
                    sudo rm -f ${TOMCAT_HOME}/webapps/${WAR_NAME}

                    # Copy new WAR
                    sudo cp target/*.war ${TOMCAT_HOME}/webapps/${WAR_NAME}

                    # Set ownership
                    sudo chown tomcat:tomcat ${TOMCAT_HOME}/webapps/${WAR_NAME}

                    echo "Restarting Tomcat..."
                    sudo systemctl restart tomcat

                    echo "Waiting for Tomcat..."
                    sleep 15

                    echo "Checking Tomcat status..."
                    sudo systemctl is-active --quiet tomcat

                    echo "Checking PetClinic application..."
                    curl -f http://localhost:${TOMCAT_PORT}/${APP_NAME}/

                    echo "========================================="
                    echo "Deployment successful!"
                    echo "Application: ${APP_NAME}"
                    echo "URL: http://SERVER-IP:${TOMCAT_PORT}/${APP_NAME}/"
                    echo "========================================="
                '''
            }
        }
    }

    post {
        success {
            echo "Build #${env.BUILD_NUMBER} succeeded. ${APP_NAME} deployed successfully to Tomcat."
        }

        failure {
            echo "Build #${env.BUILD_NUMBER} failed. Check the console output above."
        }
    }
}
```
