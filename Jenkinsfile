properties([
        pipelineTriggers([
                [$class: 'GitHubPushTrigger'], pollSCM('*/1 * * * *')
        ]),
		disableConcurrentBuilds()
])

node {
	ws("workspace/${env.JOB_NAME}/${env.BUILD_NUMBER}".replace('%2F', '_')) {
		def PROJECT_NAME;
		def ARTIFACT_VERSION;
		def DOCKER_IMAGE_NAME;
		def GIT_COMMIT_ID;
		def GIT_BRANCH_NAME;
		def GIT_REPOSITORY_URL;
		def TIMESTAMP;

		def DOCKER_IMAGE;

		try {
			stage("Checkout source") {
                checkout scm
                GIT_BRANCH_NAME = env.BRANCH_NAME.replaceAll("\\W", "_")
                GIT_REPOSITORY_URL = env.GIT_URL
                TIMESTAMP = new java.text.SimpleDateFormat("yyMMddHHmmss").format(new Date())
                IS_MASTER_BRANCH = env.BRANCH_NAME == "master"
                GIT_COMMIT_ID = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
            }

            stage("Create version number") {
                if (IS_MASTER_BRANCH) {
                    ARTIFACT_VERSION = "master.${env.BUILD_NUMBER}-${TIMESTAMP}".trim()
                } else {
                    ARTIFACT_VERSION = "${GIT_BRANCH_NAME}.${env.BUILD_NUMBER}-${TIMESTAMP}".trim()
                }
            }

            stage("Build") {
                PROJECT_NAME = "http-https-echo";


                DOCKER_IMAGE_NAME = PROJECT_NAME + ":" + ARTIFACT_VERSION;

                DOCKER_IMAGE = docker.build(
                    DOCKER_IMAGE_NAME,
                    [
                        "-f Dockerfile",
                        "."
                    ].join(" ")
                )
            }

            stage("Publish version") {
                // docker
                docker.withRegistry(env.PRIVATE_DOCKER_REGISTRY_URL, 'DOCKER_REGISTRY_USER') {
                    DOCKER_IMAGE.push()
                    dockerFingerprintFrom dockerfile: 'Dockerfile', image: DOCKER_IMAGE_NAME
                }

                // jenkins
                currentBuild.displayName = ARTIFACT_VERSION
            }
		} finally {
			stage("Clean directory"){
                deleteDir();
            }
		}
	}
}
