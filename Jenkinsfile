pipeline {

    agent any 

    environment {
    IMAGE_NAME = "faisalkhan35/spring-boot-app"
    TAG = "$BUILD_NUMBER"
    NEXUS_CREDS = credentials('nexus-creds')
    DOCKERHUB_CREDS = credentials('dockerhub-creds')
    }

    tools {
        maven 'Maven17'
    }
 
    parameters {
        choice(name: 'ENV', choices: ['Dev', 'QA', 'UAT', 'Production'], description: 'select the environment for deployment')
    }

    stages {

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar-scanners') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.2.0.4988:sonar'
                }
            }
        }

        stage('Nexus Upload') {
            steps {
                sh 'curl -v -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW --upload-file target/Spring-Boot-application.-0.0.1-SNAPSHOT.jar http://98.70.59.20:8081/repository/maven-snapshots/com/example/Spring-Boot-application./0.0.1-SNAPSHOT/Spring-Boot-application.-0.0.1-SNAPSHOT.jar'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${TAG} .'
            }
        }

        stage('Docker Hub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
            }
        }

        stage('Docker Push') {
            steps {
                sh 'docker push ${IMAGE_NAME}:${TAG}'
            }
        }

        stage('Deploy to Dev') {
            when {
                expression {
                    params.ENV == "Dev"
                }
            }

            steps {
                sh '''
                kubectl create deployment spring-boot-app --image=${IMAGE_NAME}:${TAG}
                kubectl expose deployment spring-boot-app --port=80 target-port=8080 --type=LoadBalancer
                '''
            }
        }

        stage('Deploy to QA') {
            when {
                expression {
                    params.ENV == "QA"
                }
            }

            steps {
                sh '''
                kubectl set image deployment/spring-boot-app spring-boot-app=${IMAGE_NAME}:${TAG}
                '''
            }
        }

        stage('Approval for UAT') {
            when {
                expression {
                    params.ENV == "UAT"
                }
            }

            steps {
                input message: "Approval for UAT"
            }
        }

        stage('Deploy to UAT') {
            when {
                expression {
                    params.ENV == "UAT"
                }
            }

            steps {
                sh '''
                kubectl set image deployment/spring-boot-app spring-boot-app=${IMAGE_NAME}:${TAG}
                '''
            }
        }

        stage('Change Request for Production') {
            when {
                expression {
                    params.ENV == "Production"
                }
            }

            steps {
                input message: "Is Change Request is Approved ?"
            }
        }

        stage('Approval for Production') {
            when {
                expression {
                    params.ENV == "Production"
                }
            }

            steps {
                input message: "Approval for Production"
            }
        }

        stage('Deploy to Production') {
            when {
                expression {
                    params.ENV == "Production"
                }
            }

            steps {
                sh '''
                kubectl set image deployment/spring-boot-app spring-boot-app=${IMAGE_NAME}:${TAG}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Checking Application Health Check"
                sleep 15
                curl --fail http://app || exit 1
                echo "Application is Alive Now"
                '''
            }
        }
    }

    post {
    
        success {
            sh '''
            echo "Deployment is Successful"
            '''
        }

        failure {
            sh '''
            echo "Deployment is Failed Rollback is Initiated"
            kubectl rollout undo deployment/spring-boot-app
            '''
        }
    }
}
