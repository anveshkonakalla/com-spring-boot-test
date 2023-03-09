pipeline{
    agent any
    tools {
        maven "myMaven"
    }
    stages {
        stage('Fetch code'){
            steps{
                git branch: 'master', url: 'https://github.com/anveshkonakalla/com-spring-boot-test.git'
            }
        }
        stage('Build'){
            steps{
                sh 'mvn install -DskipTests'
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
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: 'maven-test-repo',
                    credentialsId: 'nexuslogin',
                    artifacts: [
                        [artifactId: 'com-spring-boot-test',
                        classifier: '',
                        file: 'target/com-spring-boot-test-0.0.1-SNAPSHOT.jar',
                        type: 'jar']
                    ]
                )
            }
        }
    }
}
