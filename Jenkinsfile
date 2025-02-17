podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    serviceAccount: 'cicd-jenkins',
    containers: [
        containerTemplate(
            name: 'golang', 
            image: 'golang:1.10-alpine',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm', 
            image: 'alpine/helm:2.14.0',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
	environment {
	    registry = 'xiduan/hello'
	    registryCredential = 'dockerhub'
	    dockerImage = ''
	}

        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        stage ('Build') {
            container ('golang') {
                sh 'CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .'
            }
        }
        stage ('Docker') {
            container ('docker') {
		dockerImage = docker.build("xiduan/hello:${commitId}")
		docker.withRegistry('', 'dockerhub') {
	            dockerImage.push()
		}
            }
        }

        stage ('Deploy') {
            container ('helm') {
                sh "helm init --client-only --skip-refresh"
		sh "helm version"
	        sh "helm upgrade --install --wait --set image.repository=xiduan/hello,image.tag=${commitId} hello hello"
            }
        }
    }
}
