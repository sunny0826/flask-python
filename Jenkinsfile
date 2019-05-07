podTemplate(label: 'jnlp-slave', cloud: 'kubernetes',
  containers: [
    containerTemplate(
        name: 'jnlp',
        image: 'guoxudongdocker/jenkins-slave',
        alwaysPullImage: true
    ),
		containerTemplate(name: 'kubectl', image: 'guoxudongdocker/kubectl:v1.14.1', command: 'cat', ttyEnabled: true),
  ],
  nodeSelector:'ci=jenkins',
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    hostPathVolume(mountPath: '/usr/bin/docker', hostPath: '/usr/bin/docker'),
		hostPathVolume(mountPath: '/usr/local/jdk', hostPath: '/usr/local/jdk'),
    hostPathVolume(mountPath: '/usr/local/maven', hostPath: '/usr/local/maven'),
		secretVolume(mountPath: '/home/jenkins/.kube', secretName: 'devops-ctl'),
  ],
)
{
    node("jnlp-slave"){
        stage('Git Checkout'){
					git branch: '${branch}', url: 'https://github.com/sunny0826/flask-python.git'
				}
				stage('Build and Push Image'){
					withCredentials([usernamePassword(credentialsId: 'docker-register', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
						sh '''
						docker login -u ${dockerUser} -p ${dockerPassword}
						docker build -t guoxudongdocker/flask-python:${Tag} .
						docker push guoxudongdocker/flask-python:${Tag}
						'''
					}
				}
				stage('Deploy to K8s'){
					if ('true' == "${deploy}") {
						container('kubectl') {
							sh '''
							cd deploy/base
							kustomize edit set image guoxudongdocker/flask-python:${Tag}
							'''
							echo "部署到 Kubernetes"
							if ('prod' == "${ENV}") {
								sh '''
								# kustomize build deploy/overlays/prod | kubectl apply -f -
								kubectl applt -k deploy/overlays/prod
								'''
							}else {
								sh '''
								# kustomize build deploy/overlays/dev | kubectl apply -f -
								kubectl applt -k deploy/overlays/dev
								'''
							}	
						}
					}else{
						echo "跳过Deploy to K8s"
					}

				}
    }
}

