pipeline{
    agent any
    tools {
        maven "myMaven"
    }
    stages {
        stage('Build Setup'){
            steps{
               script {
                    	BRANCH_NAME=sh(script: "echo $env.GIT_BRANCH | sed -e 's|origin/||g'", returnStdout:true).trim()
                    	echo "Branch name: ${BRANCH_NAME}" 
                    	if("${BRANCH_NAME}" == 'master'){
                    		echo "Branch name: ${BRANCH_NAME}"
                    		BUILD_VERSION='2023.99.0'
                    		echo "BUILD_VERSION: ${BUILD_VERSION}"
                    	} else {
                    		BUILD_VERSION = sh(script: "echo $env.GIT_BRANCH | sed -e 's|origin/||g'", returnStdout:true).trim()
                    		echo "BUILD_VERSION: ${BUILD_VERSION}"
                    	}
                    	GIT_COMMIT= sh([script: "git rev-parse HEAD", returnStdout:true]).trim()
                        NEXUS_VERSION=sh([script: "git rev-list ${GIT_COMMIT} --count", returnStdout:true]).trim()
                        GIT_SIMPLE=sh([script: "git rev-list ${GIT_COMMIT} | head -n 1| cut -c 1-5", returnStdout:true]).trim()
                        currentBuild.displayName="${BUILD_VERSION}.${NEXUS_VERSION}-${GIT_SIMPLE}"
               }
            }
        }
        stage('Build'){
            steps{
                sh "mvn install versions:set -DnewVersion=${BUILD_VERSION}.${NEXUS_VERSION}-${GIT_SIMPLE}-SNAPSHOT -B -DskipTests"
            }
            post{
                success{
                    echo "====++++ Now archiving it ++++===="
                    archiveArtifacts artifacts: '**/target/*.jar'
                }
            }
        }
        stage('Unit Test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Checkstyle Analasis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('Sonar Analasis'){
            environment {
                scannerHome = tool 'sonar4.8'
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=com-spring-boot-test \
                   -Dsonar.projectName=com-spring-boot-test \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/example/test/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage('Quality Gate'){
            steps {
                timeout(time: 1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Artificat Upload') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'localhost:8082',
                    groupId: 'com.example',
                    version: "${BUILD_VERSION}.${NEXUS_VERSION}-${GIT_SIMPLE}",
                    repository: 'maven-test-repo',
                    credentialsId: 'nexuslogin',
                    artifacts: [
                        [artifactId: 'com-spring-boot-test',
                        classifier: '',
                        file: "target/com-spring-boot-test-${BUILD_VERSION}.${NEXUS_VERSION}-${GIT_SIMPLE}-SNAPSHOT.jar",
                        type: 'jar']
                    ]
                )
            }
        }
    }
}
