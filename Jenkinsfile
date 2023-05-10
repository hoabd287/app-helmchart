// git repository info
def gitRepository = 'https://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@github.com/hoabd287/nodejs-helm-demo.git'
def gitBranch = 'master'

def appConfigRepo = 'https://github.com/rockman88v/app-helmchart.git'
def appConfigBranch = 'master'
def helmRepo = "app-helmchart"
def helmChart = "app-demo"
def helmValueFile = "app-demo/app-demo-value.yaml"

// Image infor in registry
def imageGroup = 'hoabd4'
def appName = "nodejs-helm-app"
def namespace = "demo"

// registry credentials
def dockerhubCredential = 'jenkins_dockerhub'

dockerBuildCommand = './'
def version = "v${BUILD_NUMBER}"

pipeline {
	agent any
	
	environment {
		DOCKER_REGISTRY = 'https://registry.hub.docker.com'
		DOCKER_IMAGE_NAME = "${imageGroup}/${appName}"
		DOCKER_IMAGE = "registry.hub.docker.com/${DOCKER_IMAGE_NAME}"
	}

  stages {
  
    stage('Checkout project') 
    {
      steps 
      {
      echo "checkout project"
      git branch: gitBranch,
          url: gitRepository
      sh "git reset --hard"				
      }
    }
    stage('Build project') 
    {
      steps 
      {
      sh "npm install"
      }
    }
    stage('Build docker and push to registry') 
    {
      steps 
      {
      script {
          app = docker.build(DOCKER_IMAGE_NAME, dockerBuildCommand)
          docker.withRegistry( DOCKER_REGISTRY, dockerhubCredential ) {                       
              app.push(version)
          }

          sh "docker rmi ${DOCKER_IMAGE_NAME} -f"
          sh "docker rmi registry.hub.docker.com/hoabd4/nodejs-helm-app:${version} -f"				
        }
      }
    }
    stage('Update value in helm-chart') {
        steps {
          withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
          sh """#!/bin/bash
            [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
            git clone ${appConfigRepo} --branch ${appConfigBranch}
            cd ${helmRepo}
            sed -i 's|  tag: .*|  tag: "${version}"|' ${helmValueFile}
            git add . ; git commit -m "Update to version ${version}";git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/rockman88v/app-helmchart.git
            cd ..
            [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
            """
				}
      }
    }
  }
}
