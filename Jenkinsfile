@Library('gx-shared-lib')
import com.gxsoftware.jenkins.Release
import com.gxsoftware.jenkins.Utilities

def utils = new Utilities(steps, env, currentBuild)
def release = new Release(steps, env, currentBuild)

echo "Executing build on branch ${env.BRANCH_NAME}"

def gitRepo = 'asm_test'

node {
	try {
		
		stage(name: "Initializing ${env.BRANCH_NAME}") {
			deleteDir();
			checkout scm;
		}

		stage('Build/Test') {
			//utils.mvn "clean install"
			bat 'mvn clean install'
		}

		stage('QA Javadoc') {
			//utils.mvn "javadoc:javadoc"
		}

		stage('QA Sonar') {
			//utils.sonar()
		}

	} catch (error) {
		stage('Mail') {
/* 			def to = emailextrecipients([
							[$class: 'CulpritsRecipientProvider'],
							[$class: 'DevelopersRecipientProvider'],
							[$class: 'RequesterRecipientProvider']
			])
			utils.mailTo "Build has failed!!!", "See ${env.BUILD_URL}", "${to}" */
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

		stage("Create release") {
			bat 'mvn clean install'
		}

		stage("Deploy to nexus") {
			//bat 'mvn clean deploy'
			def folder = ''
			def pom = read_pom "${folder}pom.xml"
        	def modules = pom.modules
        	def groupId = pom.groupId
        	def packaging = pom.packaging
        	def artifactId = pom.artifactId
        	if (groupId == null || groupId.empty) {
            	groupId = pom.parent.groupId
        	}
        	if (packaging == null || packaging.empty) {
            	packaging = "jar"
        	}

			def pom2 = read_pom "pom.xml"
        	String version = pom2.version
        	if (strip_snap_shot && version.endsWith("-SNAPSHOT")) {
        	    releaseNumber = version.substring(0, version.length() - 9)
        	} else {
        	    releaseNumber = version
        	}

        	echo "packaging ${groupId}:${artifactId}:${releaseNumber}:${packaging}"
			echo modules
		}
	}
}