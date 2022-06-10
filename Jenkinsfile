pipeline {
    agent any
    environment {
                DOCKERHUB = credentials('DOCKERHUB')
    }
    stages {
		stage('PreBuild') {
            steps {
				sh 'echo Clone repository'
                git branch: 'main', credentialsId: 'jenkins-github-key', url: 'git@github.com:disasstor/ytdl-flask.git'
			}
		}
        stage('Build') {
            steps {
                sh '''
				echo ######################
				echo #Start building image#
				echo ######################
				docker build -t ilyatrof/ytdl-flask:v${BUILD_NUMBER} .
				echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
				docker push ilyatrof/ytdl-flask:v${BUILD_NUMBER}
				'''
            }
        }
        stage('Deploy') {
            steps {
				sh '''
				echo #######################
				echo #Deploy to remote host#
				echo #######################
				echo Pulling image
                docker --host $DOCKER_HOST pull ilyatrof/ytdl-flask:v${BUILD_NUMBER}
				'''
				script {
                    try{
						def old_container_id_list = sh(returnStdout: true, script: "docker --host $DOCKER_HOST ps -a | grep ilyatrof/ytdl-flask | awk '{ print \$1 }'")
                        sh 'echo Container IDs'
						println old_container_id_list
                    }catch (err) {
                        sh 'echo Kill older cootainers ERROR'
                    }
                    try{
						def old_images_id_list = sh(returnStdout: true, script: "docker --host $DOCKER_HOST images | grep ilyatrof/ytdl-flask | awk '{ print \$3 }'")
                        println old_images_id_list
						println old_images_id_list.size()
						println old_images_id_list[1]
                    }catch (err) {
                        sh 'echo Remove older image ERROR'
                    }
                }
                sh 'echo Deploy new container'
                sh 'docker --host $DOCKER_HOST run --rm --name ytdl-flask-app -d -p 5000:5000 ilyatrof/ytdl-flask:v${BUILD_NUMBER}'
            }
        }
		stage('Clearing') {
            steps {
				sh '''
				echo ######################
				echo #Clearing after build#
				echo ######################
				echo Remove docker image on Jenkins server
				docker rmi ilyatrof/ytdl-flask:v${BUILD_NUMBER}
				'''
				}
			}
		}
}