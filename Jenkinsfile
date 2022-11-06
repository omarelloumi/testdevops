pipeline {
    agent any
    environment {
        PATH = "$PATH:/opt/maven/bin"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.1.200:8081"
        NEXUS_REPOSITORY = "jenkins-repo"
        NEXUS_CREDENTIAL_ID = "deploymentRepo"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
    stages {
        stage("Fetching Code From Repository") {
            steps {
                script {
                    git branch: 'master', url: 'https://github.com/omarelloumi/testdevops.git';
                }
            }
        }


       stage('Building Project'){
            steps{
                script{
                        sh 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -DskipTests'
                        
                }
            }
        }

        stage("Runing Tests with Mockito and Jacoco") {
               steps{
                    sh 'mvn test'
                    jacoco()
                }
        }

        stage("Quality Check with SonarQube") {
            steps{
                sh 'mvn sonar:sonar -Dsonar.login=admin -Dsonar.password=181JMT1247 -DskipTests'
            }
        }
        
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "* File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "* File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        
      stage("Building Docker Image") {
                steps{
                    sh 'docker build -t $DOCKERHUB_CREDENTIALS_USR/achat .'
                }
        }
        
        stage("Login to DockerHub") {
                steps{
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                }
        }
        
        stage("Push to DockerHub") {
                steps{
                    sh 'docker push $DOCKERHUB_CREDENTIALS_USR/achat'
                }
        }
        
        stage("Docker-compose") {
                steps{
                    sh 'docker-compose up -d'
                }
        }
        
    }
}
