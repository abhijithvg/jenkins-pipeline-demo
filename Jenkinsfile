pipeline {
    agent any
    stages {
        stage ('Build Project') {
            steps {
                git url: 'https://github.com/abhijithvg/samplejava'
                withMaven(maven: 'mvn3.6.0') {
                    sh 'mvn clean package'
                }
            }
            post {
                success {
                    stash includes: '**/target/*.war', name: 'app'
                }
            }
        }
        stage ('Build Image') {
            agent {
                docker {
                    image "abhijithvg/ansible-with-docker-ws"
                    args "-v /var/run/docker.sock:/var/run/docker.sock -w /etc/ansible -e 'BUILD_ID=${env.BUILD_ID}'"
                }
            }
            steps {
                sh("rm -fr ./*")
                git url: 'https://github.com/abhijithvg/jenkins-pipeline-demo'
                unstash 'app'

		// move the target folder into the tomcat folder so that it is in the context of the Dockerfile
                sh('mv target tomcat/')
                sh("ansible-playbook ansible/build-image.yml")
            }
        }
        stage ('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh("docker login -u=${DOCKER_USERNAME} -p=${DOCKER_PASSWORD}")
                    sh("docker push abhijithvg/tomcat:pipeline-${env.BUILD_ID}")
                }

                // to save space on the local Docker Engine.
                // we do not need the new image locally
                sh("docker rmi abhijithvg/tomcat:pipeline-${env.BUILD_ID}")
            }
        }
        stage ('Deploy Image') {
            steps {
		// assumption is made that the deployment deploy/ab-deploy-tomcat is already created
                sh ("kubectl set image deploy/ab-deploy-tomcat tomcat=abhijithvg/tomcat:pipeline-${env.BUILD_ID}")
            }
        }
    }
}
