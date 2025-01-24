def COLOR_MAP = [
    "SUCCESS": 'good',
    "FAILURE": 'danger',
]

pipeline{

    agent any

    environment {
        JAVA_HOME = '/opt/java/openjdk'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        S3_BUCKET_NAME = 'vproartifact'
        S3_BUCKET_REGION = 'us-east-1'
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
                    rm -rf temp-repo
                    git clone git@github.com:ZarrTech/CI-CD-project.git temp-repo
                    cp -r temp-repo/Jenkinsfile temp-repo/Vagrantfile temp-repo/app temp-repo/configs temp-repo/db temp-repo/docker-compose.yaml temp-repo/jenkins temp-repo/web .
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
        stage('')
        stage('docker build'){
            steps{
                script{
                docker.build('vproapp', './app')
                }
            }
        }
        stage('upload to s3'){
            steps{
              script{
                try {
                    withAWS(credentials: 's3Credentials', region: '${S3_BUCKET_REGION}')
                    echo 'uploading to s3'
                    sh"""
                      # define bucket name and artifact
                       BUCKET_NAME=vproartifact
                       ARTIFACT_PATH=target/vprofile-v2.war

                       # ensure the artifact exit
                       if [! -f "$ARTIFACT_PATH"];
                       then
                         echo "artifact not found: $ARTIFACT_PATH"
                         exit 1
                       fi

                       # upload artifact to s3
                       aws s3 cp $ARTIFACT_PATH s3://$BUCKET_NAME  
                    """
                }
                catch (exception e) {
                    echo "error occured during artifact upload: ${e.message}"
                    currentBuild.result = 'FAILURE'
                    error('stopping the pipline due to s3 upload failure.')
                }
              }
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