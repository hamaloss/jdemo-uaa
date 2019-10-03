#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        sh "java -version"
    }

    stage('clean') {
        sh "chmod +x mvnw"
        sh "./mvnw -ntp clean"
    }

    stage('backend tests') {
        try {
            sh "./mvnw -ntp verify"
        } catch(err) {
            throw err
        } finally {
            junit '**/target/test-results/**/TEST-*.xml'
        }
    }

    stage('build image') {
        sh "./mvnw package -Pprod -DskipTests jib:dockerBuild"
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }

    stage('push image') {
	withCredentials([usernamePassword(credentialsId: 'region-harbor', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
		sh "docker login -u '$USERNAME' -p $PASSWORD harbor.teco.1-4.fi.teco.online"
		sh "docker tag uaa:latest harbor.teco.1-4.fi.teco.online/jdemo/uaa:latest"
		sh "docker push harbor.teco.1-4.fi.teco.online/jdemo/uaa:latest"
	}
    }
}
