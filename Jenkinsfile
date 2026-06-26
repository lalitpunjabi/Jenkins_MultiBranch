pipeline {

    agent {
        label 'DevServer'
    }

    parameters {
        choice(
            name: 'select_environment',
            choices: ['dev', 'prod']
        )
    }

    environment {
        NAME = "Lalit"
    }

    tools {
        maven 'My Maven'
    }

    stages {

        stage('build') {
            steps {
                script {
                    def file = load "script.groovy"
                    file.hello()
                }

                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('test') {

            parallel {

                stage('testA') {
                    steps {
                        echo "This is test A"
                        sh 'mvn test'
                    }
                }

                stage('testB') {
                    steps {
                        echo "This is test B"
                        sh 'mvn test'
                    }
                }
            }

            post {
                success {
                    dir("webapp/target") {
                        stash name: "maven-build", includes: "*.war"
                    }
                }
            }
        }

        stage('deploy_dev') {

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
                cp *.war /var/www/html/webapp.war

                cd /var/www/html

                rm -rf WEB-INF META-INF *.html

                jar -xvf webapp.war

                rm -f webapp.war
                '''
            }
        }

        stage('deploy_prod') {

            when {
                beforeAgent true
                branch 'master'
            }

            agent {
                label 'ProdServer'
            }

            steps {

                timeout(time: 5, unit: 'DAYS') {
                    input message: 'Deployment approved?'
                }

                unstash 'maven-build'

                sh '''
                cp *.war /var/www/html/webapp.war

                cd /var/www/html

                rm -rf WEB-INF META-INF *.html

                jar -xvf webapp.war

                rm -f webapp.war
                '''
            }
        }
    }
}
