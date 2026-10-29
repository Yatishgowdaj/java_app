pipeline {
    agent any

    tools {
        maven 'maven-3.9.2'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '2'))
    }

    environment {
        TOMCAT_URL = 'http://16.4.25.31:8080/'
        CONTEXT_PATH = '/calculator'
    }

    stages {

        stage('Code Checkout with checkout') {
            steps {
                checkout([$class: 'GitSCM',
                        branches: [[name: 'main']],
                        userRemoteConfigs: [[url: 'https://github.com/Yatishgowdaj/java_app.git',
                        credentialsId: '2b3039ac-529c-48db-9b3f-23f09543e14f']]])
            }
        }

        stage('Unit Testing') {
            steps {
                sh '''
                    cd ./calculator_app/
                    mvn clean test
                '''
            }
        }

        stage('Integration Test') {
            steps {
                dir('./calculator_app') {
                    sh 'mvn integration-test'
                }
            }
        }

        stage('Performance Test - JMeter') {
            steps {
                dir('./calculator_app') {
                    sh 'mvn verify'
                }
            }
        }

        stage('Build Package') {
            steps {
                sh '''
                    cd ./calculator_app/
                    mvn package
                '''
            }
        }

        stage('Deploy-Tomcat') {
            input {
                message "Do you want to deploy application to Tomcat10?"
                parameters {
                    choice(name: 'DEPLOY_CHOICE', choices: ['yes', 'no'])
                }
            }
            steps {
                script {
                    if ( DEPLOY_CHOICE == 'yes') {
                        echo "Deploying to Tomcat10: $TOMCAT_URL"
                        deploy adapters: [tomcat9(credentialsId: 'tomcat-manger', path: '', url: "${env.TOMCAT_URL}")],
                                         contextPath: "${env.CONTEXT_PATH}",
                                         war: 'calculator_app/target/calculator.war'
                    } else {
                        echo "Skipped deployment to Tomcat10: $TOMCAT_URL"
                    }
                }
            }
        }

    }

    post {
        always {
            // cleanWs()
            publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'calculator_app/target/jmeter/reports/CalculatorTestPlan',
                reportFiles: 'index.html',
                reportName: 'JMeter Report',
                ])

            sh '''
                ls -lrt
                tree
            '''
        }
    }

}
