pipeline {
    agent { label 'java' }
    parameters {
        booleanParam(name: 'RELEASE', defaultValue: false, description: 'Perform release?')
        string(name: 'RELEASE_VERSION', defaultValue: '', description: 'Release version')
        string(name: 'DEVELOPMENT_VERSION', defaultValue: '', description: 'Development version (without SNAPSHOT)')
    }
    stages {
        stage('Build') {
            steps {
                sh './mvnw -Dmaven.test.failure.ignore=true clean verify'
            }
        }
        stage('Archive') {
            steps {
                archiveArtifacts 'target/*.jar'
            }
        }
        stage('Release') {
            when { expression { return params.RELEASE } }
            steps {
                configFileProvider([configFile(fileId: 'bintray-settings.xml', variable: 'SETTINGS')]) {
                    sshagent(['qameta-ci_ssh']) {
                        sh 'git checkout master && git pull origin master'
                        sh "./mvnw release:prepare release:perform -B -s ${env.SETTINGS} " +
                                "-DreleaseVersion=${params.RELEASE_VERSION} " +
                                "-DdevelopmentVersion=${params.DEVELOPMENT_VERSION}-SNAPSHOT"
                    }
                }
            }
        }
    }
    post {
        failure {
            slackSend message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} failed (<${env.BUILD_URL}|Open>)",
                    color: 'danger', teamDomain: 'qameta', channel: 'allure', tokenCredentialId: 'allure-channel'
        }
    }
}
