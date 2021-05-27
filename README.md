# nexus-installation

# Pre-Requisites
    - Create EC2-Instance with t2.xlarge
    - Install Java
# Install Java
    yum install java-1.8.0-openjdk-devel -y
    cd /opt
# Download and Extract nexus tar file:
    wget https://sonatype-download.global.ssl.fastly.net/nexus/3/nexus-3.0.2-02-unix.tar.gz
    tar xvzf nexus-3.0.2-02-unix.tar.gz
# Change name of Nexus file:
    mv nexus-3.0.2-02/ nexus-1
# Add nexus user:
    useradd nexus
# Change owner ship for nexus file:
    chown -R nexus:nexus nexus-1
# Open /opt/nexus/bin/nexus.rc file and update data like as below:
    vi /opt/nexus-1/bin/nexus.rc
    -------------------------------
    run_as_user="nexus"
    -------------------------------
# Create softlink for nexus file
    ln -s opt/nexus-1/bin/nexus /etc/init.d/nexus
# Login to nexus user:
    su - nexus
# Start nexus:
    service nexus start
# Check status of nexus:
    service nexus status
# Open 8081 port in security gropus of server and Open nexus UI using ip-address with port as shown in below
    <ip-address>:8081
# Login to nexus using below credentials
    username: admin
    password: admin123
  ![image](https://user-images.githubusercontent.com/58024415/100498062-95e29d00-3185-11eb-84b2-c8bb328ed243.png)


Jenkins with Nexus:
=================

For Jenkins server open new EC2 instance and install 
    - Java
    - Git
    - Maven
    - Start jenkins after installation of Jenkin
        
            wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
            sudo rpm --import http://pkg.jenkins.io/redhat-stable/jenkins.io.key     --jenkins repo key
            sudo yum install jenkins -y
            sudo service jenkins start
            
            Password : cat /var/lib/jenkins/secrets/initialAdminPassword (copy from jenkin site after installation)
            
 # Install 'pipeline-utility-steps' plugin
 
 Manage Jenkins -->Manage Plugin--->Pipeline utility steps -install without restart
 
 Manage Plugin -> GitLab Plugin and Maven Integration Plugin
 
 Under Global Tool configuration 
 
 ![Capture](https://user-images.githubusercontent.com/54719289/103825632-34bebb00-509b-11eb-8daf-4ed2f0bb496e.JPG)
 
 Require Publish to nexus repository plugin so install the plugin - 
 
 Nexus Artifact Uploader
 
 Fisrt Global credential is required

(Manage Jenkins-->Manage Credential -->select global  --Add credential

![Capture](https://user-images.githubusercontent.com/54719289/103827795-c5979580-509f-11eb-9fff-26e4c9e7d7e1.JPG)


![Capture](https://user-images.githubusercontent.com/54719289/103828062-4c4c7280-50a0-11eb-9cd0-5c251bacb516.JPG)

![Capture](https://user-images.githubusercontent.com/54719289/103828112-68501400-50a0-11eb-8ca5-0389d221c61d.JPG)

```
Script used :
pipeline {
    agent any
    tools {
        maven "maven"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.135.236.176:8081"
        NEXUS_REPOSITORY = "maven-releases"
        NEXUS_CREDENTIAL_ID = "nexus-credential"
    }
    stages {
        stage("SCM") {
            steps {
                script {
                    git 'https://github.com/Naresh240/springboohello-CICD.git';
                }
            }
        }
        stage("Maven Build") {
            steps {
                script {
                    sh "mvn package -DskipTests=true"
                }
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script{
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
    }
}

```

![Capture](https://user-images.githubusercontent.com/54719289/103832569-92a3d080-50a4-11eb-8609-fcb83ba4f5ec.JPG)


![Capture](https://user-images.githubusercontent.com/54719289/103827623-6cc7fd00-509f-11eb-991b-4dc8b3b6f619.JPG)

 
 
