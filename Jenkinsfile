pipeline {

    agent {
        label 'DevServer'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        choice(
            name: 'select_environment',
            choices: ['dev', 'prod'],
            description: 'Select deployment environment'
        )
    }

    environment {
        NAME = "piyush"
        DEPLOY_PATH = "/var/www/html"
    }

    tools {
        maven 'My Maven'
    }

    stages {

        stage('Build') {
            steps {
                script {
                    def file = load 'script.groovy'
                    file.hello()
                }

                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Test') {

            parallel {

                stage('Test A') {
                    steps {
                        echo 'Running Test A'
                        sh 'mvn test'
                    }
                }

                stage('Test B') {
                    steps {
                        echo 'Running Test B'
                        sh 'mvn test'
                    }
                }

            }

            post {
                success {
                    dir('webapp/target') {
                        stash name: 'maven-build', includes: '*.war'
                    }
                }
            }
        }

        stage('Deploy to Development') {

            when {
                beforeAgent true
                branch 'develop'
            }

            agent {
                label 'DevServer'
            }

            steps {

                unstash 'maven-build'

                sh '''
                WAR=$(ls *.war)

                echo "Deploying $WAR to Development"

                rm -rf ${DEPLOY_PATH}/*

                cp "$WAR" ${DEPLOY_PATH}/

                cd ${DEPLOY_PATH}

                jar -xvf "$WAR"

                rm -f "$WAR"

                echo "Development Deployment Successful"
                '''
            }
        }

        stage('Approval for Production') {

            when {
                beforeAgent true
                branch 'master'
            }

            steps {

                timeout(time: 5, unit: 'DAYS') {
                    input message: 'Deploy application to Production?'
                }

            }
        }

        stage('Deploy to Production') {

            when {
                beforeAgent true
                branch 'master'
            }

            agent {
                label 'ProdServer'
            }

            steps {

                unstash 'maven-build'

                sh '''
                WAR=$(ls *.war)

                echo "Deploying $WAR to Production"

                rm -rf ${DEPLOY_PATH}/*

                cp "$WAR" ${DEPLOY_PATH}/

                cd ${DEPLOY_PATH}

                jar -xvf "$WAR"

                rm -f "$WAR"

                echo "Production Deployment Successful"
                '''
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            cleanWs()
        }
    }
}
