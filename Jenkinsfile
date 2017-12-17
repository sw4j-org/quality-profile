pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                withMaven(jdk: 'Current JDK 8',
                        maven: 'Current Maven 3',
                        mavenLocalRepo: '${JENKINS_HOME}/maven-repositories/${EXECUTOR_NUMBER}/') {
                    sh "mvn clean install"
                }
            }
        }
        stage('Document and Deploy') {
            when {
                environment name: 'CHANGE_FORK', value: ''
                expression { GIT_URL ==~ 'https://github.com/sw4j-org/.*' }
            }
            parallel {
                stage('Deploy') {
                    steps {
                        withMaven(jdk: 'Current JDK 8',
                                maven: 'Current Maven 3',
                                mavenLocalRepo: '${JENKINS_HOME}/maven-repositories/${EXECUTOR_NUMBER}/') {
                            sh "mvn deploy"
                        }
                    }
                }
                stage('Create Site') {
                    steps {
                        withMaven(jdk: 'Current JDK 8',
                                maven: 'Current Maven 3',
                                mavenLocalRepo: '${JENKINS_HOME}/maven-repositories/${EXECUTOR_NUMBER}/') {
                            sh "mvn -Dscmpublish.skipCheckin=true post-site scm-publish:publish-scm"
                        }
                        withCredentials([string(credentialsId: "${TOKEN}", variable: 'GH_TOKEN')]) {
                            sh """
                                cd target/scmpublish-checkout
                                git commit -a -m 'Automatic created documentation'
                                git push -fq https://${GH_TOKEN}@github.com/sw4j-org/sandbox.git gh-pages:gh-pages
                            """
                        }
                    }
                }
            }
        }
    }
}