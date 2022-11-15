def CHROME
def APPXD
def NETWORK = UUID.randomUUID().toString()
def CHROME_CONTAINER = "chrome"
def APP_CONTAINER = "app"

pipeline {
	agent none
	stages {
		stage('bootstrap') {
			agent any
			steps {
				script {
					sh "docker network create ${NETWORK} || true"
					CHROME = docker.image('selenium/standalone-chrome:106.0').run("--name ${CHROME_CONTAINER} --network ${NETWORK} --shm-size=2g")
					APPXD = docker.build("${APP_CONTAINER}", "./").run("--network ${NETWORK} --name app")
				}
			}
		}
		stage('Integration UI Test') {
			parallel {
				stage('Headless Browser Test') {
					agent {
						docker {
							image 'maven:3-alpine' 
							args "-v /root/.m2:/root/.m2 --network ${NETWORK}"
						}
					}
					steps {
						sh 'mvn -B -DskipTests clean package'
						sh 'mvn test'
					}
				}
			}
		}
	}
	post {
		always {
			node(null) {
				script {
					if (CHROME) {
						CHROME.stop()
					}

					if (APPXD) {
						APPXD.stop()
					}
					sh "docker network rm ${NETWORK} || true"
				}
				junit 'target/surefire-reports/*.xml'	
			}
		}
	}
}