def label = "mypod-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
	containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat', privileged: 'true')
],
volumes: [
	hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
    node(label) {
		container('docker') {
			stage('Get Mendix Build Pack') {
				git 'https://github.com/mendix/docker-mendix-buildpack'
			}
			
			stage('Retrieve project from scm') {
                checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'project']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/cstevandy/test-pipeline.git']]]
			}
			
			stage('Build') {
				docker.withServer('unix:///var/run/docker.sock') {
					docker.withRegistry( 'https://registry.hub.docker.com', 'docker-credentials' )   {
						def image = docker.build("chrisgg/test-pipeline:latest", '--build-arg BUILD_PATH="./project" --build-arg ROOTFS_IMAGE=mendix/rootfs .')
						withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-credentials',
						usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
							sh 'docker login -u "$USERNAME" -p "$PASSWORD"';
							image.push();
						}
					}
				}
			}
		}
    }
}
