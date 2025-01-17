def COLOR_MAP = [
    "SUCCESS": 'good',
    "FAILURE": 'danger',
]

pipeline{

    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/temurin-11-jdk-amd64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    tools{
        maven 'maven'
        jdk 'jdk17'
    }

    stages{
        stage('clone repo'){
            steps{
                git 'https://github.com/devopshydclub/vprofile-project.git'
            }
        }
        stage('Verify JAVA_HOME') {
            steps {
                sh 'echo $JAVA_HOME'
                sh '/opt/java/openjdk/bin/java -version'
            }
}
        // stage('Unit test'){
        //     steps{
        //         sh "mvn -Djava.home=$JAVA_HOME test"
        //         sh 'mvn test'
        //     }
        // }
        stage('checkstyle'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('build'){
            steps{
                sh 'mvn clean package -X'
            }
        }

        stage('sonarqube analysis'){
            environment{
                scannerHome = tool 'sonar6.2'
            }
            steps{
                withSonarQubeEnv('sonarserver'){
                    sh"""
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    """
                }
            }
        }
        stage('upload artifact'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'http://192.168.56.20:8081',
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: 'vpro-repo',
                    credentialsId: 'nexuscred',
                    artifacts: [
                        [artifactId: 'vprofile', file: 'target/vprofile-v2.war', type: 'war']
                    ]
                    )
            }
        }

    }

    post {
        always {
            echo 'slack notification'
            slackSend channel: '#devOps-cicd',
                      color: COLOR_MAP[currentBuild.result],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n more details at ${env.BUILD_URL}"
        }
    }
}