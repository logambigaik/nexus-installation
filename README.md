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
