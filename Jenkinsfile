def COLOR_MAP = [
    "SUCCESS": 'good',
    "FAILURE": 'danger',
]

pipeline{

    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/temurin-11-jdk-amd64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        DOCKER_HUB_CREDENTIALS = 'dockerID'
    }

    tools{
        maven 'maven'
        jdk 'jdk17'
    }

    stages{
        stage('clone repo'){
            steps{
                git branch: 'atom', url: 'https://github.com/hkhcoder/vprofile-project.git'
            }
        }
        stage('Fetch Dockerfile') {
            steps {
                sh """
                    git clone git@github.com:ZarrTech/CI-CD-project.git temp-repo
                    cp temp-repo/app/Dockerfile .
                    rm -rf temp-repo
                """
            }
        }
        stage('Verify JAVA_HOME') {
            steps {
                sh 'echo $JAVA_HOME'
                sh '/opt/java/openjdk/bin/java -version'
            }
        }
        stage('build'){
            steps{
                sh 'mvn -X clean package'
            }
        }
        stage('Unit test'){
            steps{
                sh "mvn -Djava.home=$JAVA_HOME test"
                sh 'mvn test'
            }
        }
        stage('checkstyle'){
            steps{
                sh 'mvn checkstyle:checkstyle'
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
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml \
                        -Dsonar.host.url=http://192.168.56.20:9000 \
                        -Dsonar.ws.timeout=60
                    """
                }
            }
        }
        // stage('quality gate'){
        //     steps{
        //         timeout(time: 1, unit: 'HOURS'){
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }
        stage('upload artifact'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '192.168.56.20:8081',
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
        stage('docker build'){
            steps{
                script{
                docker.build('vproapp', './app/')
                }
            }
        }
        stage('deploy to app server'){
            steps{
                sh"""
                 curl -o vprofile-v2.war http://192.168.56.20:8081/repository/vpro-repo/QA/vprofile/${env.BUILD_ID}-${env.BUILD_TIMESTAMP}/vprofile-v2.war
                 docker cp vprofile-v2.war vproapp:/usr/local/tomcat/webapps/ROOT.war
                 docker restart vproapp
                """
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