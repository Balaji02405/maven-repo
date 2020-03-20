def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
    containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'kubectl', image: 'codefresh/kubectl', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]
  ) {
    node(label) {
        stage('Git pull code') {
            git 'https://github.com/balaji024/maven-project.git'
        }
        stage('Build Maven project') {
            container('maven') {
                sh 'mvn -B clean package'
            }
        }
        stage('Create Docker image') {
            container('docker') {
                withCredentials([[$class: 'UsernamePasswordMultiBinding',
                    credentialsId: 'DOCKERHUB',
                    usernameVariable: 'DOCKER_HUB_USER',
                    passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                    sh """
                        docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
                        docker build -t balaji024/webapp2:${BUILD_NUMBER} .
                        docker push balaji024/webapp2:${BUILD_NUMBER}
                        """
                    }
            }
        }
        stage('QA Approval') {
            input('ALL TESTS PASSED?')
            sh "echo 'Approved. Ready for deployment...'"
        }
	stage('Deploy Application on K8S') {
	    container('kubectl') {
		withKubeConfig([credentialsId: 'KUBEADMIN1', serverUrl: 'https://172.25.4.66:8443', namespace: 'balajijenkins']) {
		    sh """
			kubectl run webapp${BUILD_NUMBER} --image=balaji024/webapp2:${BUILD_NUMBER} -n balajijenkins
			kubectl expose deploy webapp${BUILD_NUMBER} --port=8080 --target-port=8080 --type=NodePort --name=webapp${BUILD_NUMBER} -n balajijenkins
			"""
		}
	    }
	}
    }
  }
