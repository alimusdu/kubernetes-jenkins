#!/usr/bin/groovy

// load pipeline functions
// Requires pipeline-github-lib plugin to load library from github
@Library('github.com/lachie83/jenkins-pipeline@master')
def pipeline = new io.estrado.Pipeline()

podTemplate(label: 'jenkins-pipeline', containers: [
    ontainerTemplate(name: 'jnlp',
                        image: 'jenkins/inbound-agent:4.13-2-alpine',
                        runAsUser: '0',
                        resourceRequestCpu: '1',
                        resourceLimitCpu: '1',
                        resourceRequestMemory: '1Gi',
                        resourceLimitMemory: '1Gi'),
    containerTemplate(name: 'docker', image: 'docker:25.0.0-cli', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'maven', image: 'maven:3.9.6', command: 'cat', ttyEnabled: true),
],
volumes:[
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
]){

  node ('jenkins-pipeline') {

    def pwd = pwd()
    def chart_dir = "${pwd}/charts/hellojava"
    def tags = [env.BUILD_TAG, 'latest']
    def docker_registry_url = "jcorioland.azurecr.io"
    def app_hostname = "hellojava.aks.jcorioland.io";
    def docker_email = "jucoriol@microsoft.com"
    def docker_repo = "hellojava"
    def docker_acct = "kubernetes"
    def jenkins_registry_cred_id = "acr_creds"

    // checkout sources
    checkout scm

    // set additional git envvars for image tagging
    pipeline.gitEnvVars()

    // Execute Maven build and tests
    stage ('Maven Build & Tests') {

      container ('maven') {
        sh "mvn install"
      }

    }
    // Build and push the Docker image
    stage ('Build & Push Docker image') {

      container('docker') {
        println "build & push"

        // perform docker login
        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: jenkins_registry_cred_id, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
          sh "docker login -e ${docker_email} -u ${env.USERNAME} -p ${env.PASSWORD} ${docker_registry_url}"
        }

        // build and publish container
        pipeline.containerBuildPub(
            dockerfile: "./",
            host      : docker_registry_url,
            acct      : docker_acct,
            repo      : docker_repo,
            tags      : tags,
            auth_id   : jenkins_registry_cred_id
        )
      }
    }
  }
}
