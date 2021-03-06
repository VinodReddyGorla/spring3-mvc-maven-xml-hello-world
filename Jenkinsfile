pipeline {
    agent {
        label "master"
    }
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "maven-3.6.3"
        jdk "jdk"
    }
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "192.168.0.102:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "maven-test1"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus_credentials"
        // tomcat credential id for ssh authentiation
        TOMCAT_CREDENTIAL_ID = "tomcat-sshpk"
        // tomcat server user name
        TOMCAT_SERVER_USER = "vinod"
        // tomcat server IP
        TOMCAT_SERVER_IP = "192.168.0.110"
        // deployment path
        PATH_WEBAPPS = "/opt/tomcat8/webapps"
        // package source path
        PACKAGE_PATH = "/root/.jenkins/workspace/maven-project/target/*.war"
    }
    stages {
        stage("clone code") {
            
            steps {
                    // Let's clone the source
                    git 'https://github.com/VinodReddyGorla/spring3-mvc-maven-xml-hello-world.git';
            }
        }
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                  //  if (env.BRANCH_NAME == "master") {
                          echo " packaging master branch" 
                          sh 'mvn -Dmaven.test.failure.ignore=true clean package'
                    }
                  //  else {
                     //   currentBuild.result = 'ABORTED'
                    //    echo "this is not a master branch" 
                    //}
                //}
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                          //  version: '${BUILD_NUMBER}',
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact  generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage ('deploy'){
            steps {
                
                sshagent(credentials : ['TOMCAT_CREDENTIAL_ID']) {
                    sh 'ssh -o StrictHostKeyChecking=no ${TOMCAT_SERVER_USER}@${TOMCAT_SERVER_IP} uptime'
                    sh 'ssh -v ${TOMCAT_SERVER_USER}@${TOMCAT_SERVER_IP}'
                    sh 'rm -rf  ${PATH_WEBAPPS}/*.war '
                    echo " old packages removed "
                    sh 'scp ${PACKAGE_PATH} ${TOMCAT_SERVER_USER}@${TOMCAT_SERVER_IP}:${PATH_WEBAPPS}'
                    echo "package succesfully moved to webapps folder in remote server"

        }
                
            }
        }
    }
}
