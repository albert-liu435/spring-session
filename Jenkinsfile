properties([
		buildDiscarder(logRotator(numToKeepStr: '10')),
		pipelineTriggers([
				cron('@daily')
		]),
])

def SUCCESS = hudson.model.Result.SUCCESS.toString()
currentBuild.result = SUCCESS

def GRADLE_ENTERPRISE_CACHE_USER = usernamePassword(credentialsId: 'gradle_enterprise_cache_user',
		passwordVariable: 'GRADLE_ENTERPRISE_CACHE_PASSWORD',
		usernameVariable: 'GRADLE_ENTERPRISE_CACHE_USERNAME')
def GRADLE_ENTERPRISE_SECRET_ACCESS_KEY = string(credentialsId: 'gradle_enterprise_secret_access_key',
		variable: 'GRADLE_ENTERPRISE_ACCESS_KEY')
def SPRING_SIGNING_SECRING = file(credentialsId: 'spring-signing-secring.gpg', variable: 'SIGNING_KEYRING_FILE')
def SPRING_GPG_PASSPHRASE = string(credentialsId: 'spring-gpg-passphrase', variable: 'SIGNING_PASSWORD')
def OSSRH_CREDENTIALS = usernamePassword(credentialsId: 'oss-token', passwordVariable: 'OSSRH_PASSWORD', usernameVariable: 'OSSRH_USERNAME')
def ARTIFACTORY_CREDENTIALS = usernamePassword(credentialsId: '02bd1690-b54f-4c9f-819d-a77cb7a9822c', usernameVariable: 'ARTIFACTORY_USERNAME', passwordVariable: 'ARTIFACTORY_PASSWORD')
def JENKINS_PRIVATE_SSH_KEY = file(credentialsId: 'docs.spring.io-jenkins_private_ssh_key', variable: 'DEPLOY_SSH_KEY')
def SONAR_LOGIN_CREDENTIALS = string(credentialsId: 'spring-sonar.login', variable: 'SONAR_LOGIN')

def jdkEnv(String jdk = 'jdk8') {
	def jdkTool = tool(jdk)
	return "JAVA_HOME=${ jdkTool }"
}

parallel check: {
    stage('Check') {
        timeout(time: 45, unit: 'MINUTES') {
            node('linux') {
                label 'spring-session'
                checkout scm
                sh "git clean -dfx"
                try {
                    withCredentials([GRADLE_ENTERPRISE_CACHE_USER,
                         GRADLE_ENTERPRISE_SECRET_ACCESS_KEY,
                         ARTIFACTORY_CREDENTIALS]) {
                        withEnv([jdkEnv(),
                             "GRADLE_ENTERPRISE_CACHE_USERNAME=${GRADLE_ENTERPRISE_CACHE_USERNAME}",
                             "GRADLE_ENTERPRISE_CACHE_PASSWORD=${GRADLE_ENTERPRISE_CACHE_PASSWORD}",
                             "GRADLE_ENTERPRISE_ACCESS_KEY=${GRADLE_ENTERPRISE_ACCESS_KEY}"]) {
                            sh "./gradlew check -PartifactoryUsername=$ARTIFACTORY_USERNAME -PartifactoryPassword=$ARTIFACTORY_PASSWORD --stacktrace"
                        }
                    }
                }
                catch (e) {
                    currentBuild.result = 'FAILED: check'
                    throw e
                }
                finally {
                    junit '**/build/test-results/*/*.xml'
                }
            }
        }
    }
},
jdk11: {
    stage('JDK 11') {
        timeout(time: 45, unit: 'MINUTES') {
            node('linux') {
                checkout scm
                sh "git clean -dfx"
                try {
                    withCredentials([GRADLE_ENTERPRISE_CACHE_USER,
                         GRADLE_ENTERPRISE_SECRET_ACCESS_KEY,
                         ARTIFACTORY_CREDENTIALS]) {
                        withEnv([jdkEnv("jdk11"),
                         "GRADLE_ENTERPRISE_CACHE_USERNAME=${GRADLE_ENTERPRISE_CACHE_USERNAME}",
                         "GRADLE_ENTERPRISE_CACHE_PASSWORD=${GRADLE_ENTERPRISE_CACHE_PASSWORD}",
                         "GRADLE_ENTERPRISE_ACCESS_KEY=${GRADLE_ENTERPRISE_ACCESS_KEY}"]) {
                            sh "./gradlew test integrationTest -PartifactoryUsername=$ARTIFACTORY_USERNAME -PartifactoryPassword=$ARTIFACTORY_PASSWORD --stacktrace"
                        }
                    }
                }
                catch (e) {
                    currentBuild.result = 'FAILED: jdk11'
                    throw e
                }
            }
        }
    }
},
jdk12: {
    stage('JDK 12') {
        timeout(time: 30, unit: 'MINUTES') {
            node('linux') {
                checkout scm
                sh "git clean -dfx"
                try {
                    withCredentials([GRADLE_ENTERPRISE_CACHE_USER,
                         GRADLE_ENTERPRISE_SECRET_ACCESS_KEY,
                         ARTIFACTORY_CREDENTIALS]) {
                        withEnv([jdkEnv("openjdk12"),
                         "GRADLE_ENTERPRISE_CACHE_USERNAME=${GRADLE_ENTERPRISE_CACHE_USERNAME}",
                         "GRADLE_ENTERPRISE_CACHE_PASSWORD=${GRADLE_ENTERPRISE_CACHE_PASSWORD}",
                         "GRADLE_ENTERPRISE_ACCESS_KEY=${GRADLE_ENTERPRISE_ACCESS_KEY}"]) {
                            sh "./gradlew test integrationTest -PartifactoryUsername=$ARTIFACTORY_USERNAME -PartifactoryPassword=$ARTIFACTORY_PASSWORD --stacktrace"
                        }
                    }
                }
                catch (e) {
                    currentBuild.result = 'FAILED: jdk12'
                    throw e
                }
            }
        }
    }
}
