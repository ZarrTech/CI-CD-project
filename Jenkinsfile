def COLOR_MAP = [
    "SUCCESS": 'good',
    "FAILURE": 'danger',
]

pipeline{

    agent any

    tools{
        maven 'maven'
        jdk 'jdk17'
    }

    stages{
        stage('clone repo'){
            steps{
                git 'https://github.com/devopshydclub/vprofile-repo.git'
            }
        }
        stage('Unit test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('checkstyle'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }stage('build'){
            steps{
                sh 'mvn clean package'
            }
        }

        stage('sonarqube analysis'){
            environment{
                scannerHome = tool 'sonar6.2'
            }
            steps{
                withSonarQubeEnv('sonarserver'){
                    sh"""${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=vprofile \ 
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/\
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/\
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml"""
                }
            }
        }
        stage('upload artifact'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'http://http://192.168.56.20:8081',
                    groupId: 'QA',
                    version: ${env.BUILD_ID}-${env.BUILD_TIMESTAMP},
                    repository: 'vpro-repo',
                    credentialsId: 'nexuscred',
                    artifacts: [
                        [artifactId: 'vprofile', file: 'target/vprofile-v2.war', type: 'war']
                    ]
                    )
            }
        }

        // stage('build docker image'){
        //     sh 'docker compose build -t app ./app'
        // }
    }

    post('Always run'){
        always{
            echo 'slack notification'
            slackSend channel: '#devOpscicd',
            color: COLOR_MAP[currentBuild.result],
            message:"*${currentBuild.ccurrentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n more details at ${env.BUILD_URL}",
        }
    }
}