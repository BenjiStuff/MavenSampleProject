@Library('gx-shared-lib')
import com.gxsoftware.jenkins.Release
import com.gxsoftware.jenkins.Utilities

def utils = new Utilities(steps, env, currentBuild)
def release = new Release(steps, env, currentBuild)

echo "Executing build on branch ${env.BRANCH_NAME}"

def gitRepo = 'asm_test'

node {
	try {
		
		stage(name: "initializing ${env.BRANCH_NAME}") {
			deleteDir();
			checkout scm;
		}

		stage('Build/Test') {
			utils.mvn "clean install"
		}

		stage('QA Javadoc') {
			utils.mvn "javadoc:javadoc"
		}

		stage('QA Sonar') {
			utils.sonar()
		}

	} catch (error) {
		stage('Mail') {
			def to = emailextrecipients([
							[$class: 'CulpritsRecipientProvider'],
							[$class: 'DevelopersRecipientProvider'],
							[$class: 'RequesterRecipientProvider']
			])
			utils.mailTo "Build has failed!!!", "See ${env.BUILD_URL}", "${to}"
		}
		throw error;
	}
}

if (env.BRANCH_NAME == 'develop') {
	node {
		stage("Prepare dependency management") {
			utils.mvn "clean install -DskipTests"
		}

		stage("QA Dependency Updates") {
			dependencyUpdates utils
		}

		stage("QA Dependency Security") {
			dependencySecurity utils
		}
	}
}
if (env.BRANCH_NAME == 'master') {
	def releaseNumber
	def releaseTag
	def developmentVersion

	node {
		stage("Prepare for release") {
			echo "Testing this feature"
		}
	}

}
