# Build-Continuous-Integration-with-Jenkins-and-Dokcer

## Prerequisite
    •	Install Ubuntu and Docker  - https://docs.docker.com/engine/installation/linux/ (OR)
  
    •	Install Docker Toolbox in windows - https://docs.docker.com/docker-for-windows/
  
## Jenkins + DInD (Docker – In – Docker)

    •	This includes Docker installation inside of Jenkins.

    •	This maintains the created containers as children.

    •	This would be preferable if want to manage a complete clean Docker environment inside Jenkins.
    
## Jenkins + DOOD (Docker – Outside – Of – Docker):

    •	This uses underlying host’s Docker installation.
    
    •	This maintains the created containers as siblings.
    
    Advantages:
    
        •	Enables sharing of images with host OS
        
              o	Eliminates storing images multiple times
              
              o	Makes it possible for Jenkins to automate local image creation
              
        •	Eliminates a virtualization layer (lxc)
        
        •	Any settings in the host’s Docker daemon will apply to Jenkins container as well
        
        •	Easier to set up, just need to map the host’s Docker executable and daemon socket on to the container
        
        •	Host and Jenkins container will use the same version of Docker, always.
        
        •	No privileged mode needed
        
        •	Permits the jenkins user to run docker without the sudo prefix.
        
        •	Allows greater flexibility at runtime.
        
        •	Ability to reuse the image cache from the host.
        
Note: In general, for testing and production environment, DOOD is chosen instead of DinD    

## Docker Toolbox Overview:

        1.	Once Docker Toolbox installed in windows , will be seen two shortcuts on desktop as below

![1](https://cloud.githubusercontent.com/assets/20100300/17997903/145950c6-6b37-11e6-85ea-387c4feadc5b.JPG)

        2.	Double-Click the “Docker Quick start Terminal” shortcut icon, execute the command 
        “pwd “will come to know which directory mapped on windows.

![2](https://cloud.githubusercontent.com/assets/20100300/17997906/19905a80-6b37-11e6-92fe-11b13c0a9ad0.JPG)

        3.	Create a directory for workspace by executing the command “mkdir docker-workspace”

![3](https://cloud.githubusercontent.com/assets/20100300/17997907/19beeec2-6b37-11e6-8c1b-2eea779bf4e9.JPG)

        4.	All the successive steps mentioned will be followed by considering the above mentioned directory 
        as base directory. (For instance, here base directory - /c/users/Administrator/docker-workspace) 
        
## Install Jenkins Master (with DOOD):

        1.	Create a directory “Jenkins” under “docker-workspace” directory by executing the command
        “mkdir Jenkins”
![1](https://cloud.githubusercontent.com/assets/20100300/17998183/b513716c-6b38-11e6-9170-3c8287e1ed26.JPG)

        2.	Create a directory “Master” under “Jenkins” directory by executing the command “mkdir Master”
![2](https://cloud.githubusercontent.com/assets/20100300/17998184/b540af42-6b38-11e6-8e72-b9ec8beeba99.JPG)

        3.	Create a file with name “Dockerfile” under “Master” directory and fill with the below content
![3](https://cloud.githubusercontent.com/assets/20100300/17998185/b56eba54-6b38-11e6-8ea4-1883e2db623c.JPG)

                    FROM jenkins:latest
                    MAINTAINER Kranthi Kumar Bitra <kranthi.b76@gmail.com>

                    USER root
                    RUN apt-get update \
                    && apt-get install -y sudo supervisor \
                    && rm -rf /var/lib/apt/lists/*

                    RUN curl -sSL https://get.docker.com/ | sh

                    RUN usermod -aG docker root

                    USER root
                    RUN mkdir -p /var/log/supervisor
                    RUN mkdir -p /var/log/jenkins
                    RUN curl -L "https://cli.run.pivotal.io/stable?release=linux64-binary&source=github" | tar -zx
                    RUN /cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-linux_x86 -f
                    COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
                    CMD /usr/bin/supervisord -c /etc/supervisor/conf.d/supervisord.conf
                    
        4.	Create a file with name “supervisord.conf”  under “Master” directory , fill with the below content
![4](https://cloud.githubusercontent.com/assets/20100300/17998187/b59c5414-6b38-11e6-84f7-95b8f082762e.JPG)

                    [supervisord]
                    user=root
                    nodaemon=true

                    [program:chown]
                    priority=20
                    command=chown -R root:root /var/jenkins_home

                    [program:root]
                    user=root
                    autostart=true
                    autorestart=true
                    command=/usr/local/bin/jenkins.sh
                    redirect_stderr=true
                    stdout_logfile=/var/log/jenkins/%(program_name)s.log
                    stdout_logfile_maxbytes=10MB
                    stdout_logfile_backups=10
                    environment = JENKINS_HOME="/var/jenkins_home",HOME="/var/jenkins_home",USER="root"
        
        5.	Create a file with name “plugins.txt” under “Master” directory , fill with the below content
![5](https://cloud.githubusercontent.com/assets/20100300/17998186/b59ba30c-6b38-11e6-8fdf-5284e6bc1d8c.JPG)

                    structs:latest
                    workflow-step-api:latest
                    workflow-scm-step:latest
                    scm-api:latest
                    scm-sync-configuration:latest
                    credentials:latest
                    mailer:latest
                    junit:latest
                    subversion:latest
                    script-security:latest
                    matrix-project:latest
                    ssh-credentials:latest
                    git-client:latest
                    git:latest
                    greenballs:latest
                    ssh-slaves:latest
                    token-macro:latest
                    durable-task:latest
                    docker-plugin:latest
        
        6.	Create a file with name “docker-compose.yml” under “Master” directory, fill with the below content
![6](https://cloud.githubusercontent.com/assets/20100300/17998188/b59c4014-6b38-11e6-92fa-cc78151cafad.JPG)

                    myjenkins:
                        image: myjenkins_cloudfoundary_dood 
                        volumes:
                            - /var/run/docker.sock:/var/run/docker.sock
                            - ./jenkins_home:/var/jenkins_home
                        ports:
                            - "8080:8080"
                            - "50000:50000"
        7.	In command prompt, go to the directory where the above four files exists, execute the below commands
        to start Jenkins with DOOD.
        
                a.	docker build –t myjenkins_cloudfoundary_dood .  (this will take few minutes time to build 
                the image – only one time should execute)
![7](https://cloud.githubusercontent.com/assets/20100300/17998189/b59dbc50-6b38-11e6-8a73-7404fbe5f578.JPG)

                b.	In order to confirm whether the image got created, execute the command 
                -  “ docker images myjenkins_cloudfoundary_dood”
![8](https://cloud.githubusercontent.com/assets/20100300/17998190/b59f3e2c-6b38-11e6-951c-ff9609f02697.JPG)

                c.	docker-compose up
![9](https://cloud.githubusercontent.com/assets/20100300/17998191/b5a4d45e-6b38-11e6-9651-a0830ac5a42c.JPG)

                d.	In order to confirm whether the Jenkins process created or not , execute the command 
                – “docker ps –a” which should be in “Up” status
![10](https://cloud.githubusercontent.com/assets/20100300/17998192/b5c9daec-6b38-11e6-8cc9-64a8a5868861.JPG)


## Jenkins Master Initial Configuration (One time Configuration)

    •	Access the Jenkins in windows with url – http://192.168.99.100:8080 . 
    Once this got accessed, will be seen “Unlock Jenkins” page.
![0](https://cloud.githubusercontent.com/assets/20100300/17998785/b9100d72-6b3b-11e6-8b1d-20ba984a6715.JPG)

    •	To get the administrator password, go to the “/c/users/Administrator/docker-workspace/
    Jenkins/Master/jenkins_home/secrets” directory. In that open the “initialAdminPassword” file.
![0-1](https://cloud.githubusercontent.com/assets/20100300/17998786/b982b5a2-6b3b-11e6-9aff-1b410c8fed48.JPG)

    •	Copy the content from the above file and paste in the “Administrator password” text area 
    and click “Continue” button.
![1](https://cloud.githubusercontent.com/assets/20100300/17998791/b9a0a990-6b3b-11e6-83f0-3fdf84f58afc.JPG)

    •	Now, it will redirect to “Customize Jenkins”  page and select the “Install suggested plugins”
![2](https://cloud.githubusercontent.com/assets/20100300/17998787/b99b28b2-6b3b-11e6-9ba1-69bfad3708ee.JPG)

    •	After few minutes, all the suggested plugins will be installed automatically.
![3](https://cloud.githubusercontent.com/assets/20100300/17998790/b99c3162-6b3b-11e6-99e0-8e7d535d0254.JPG)

    •	Now, it will redirect to “Create First Admin User”, enter the user details and click 
    “Save and Finish” button.
![4](https://cloud.githubusercontent.com/assets/20100300/17998792/b9a0dfb4-6b3b-11e6-935f-0ccf2805b118.JPG)

    •	Now, will get set up complete page, click the “Start using Jenkins” button.
![5](https://cloud.githubusercontent.com/assets/20100300/17998788/b99bf918-6b3b-11e6-8241-595d7803f3ea.JPG)    

    •	Click the “Manage Jenkins” link in order to install few plugins required for this Continuous 
    Integration.
![6](https://cloud.githubusercontent.com/assets/20100300/17998793/b9b2571c-6b3b-11e6-9c3e-61f865b11fb0.JPG)

    •	Now, click the “Manage Plugins” link to go to “Plugin Manager” page.
![7](https://cloud.githubusercontent.com/assets/20100300/17998794/b9c9a1d8-6b3b-11e6-9ec1-5b72f142e0c5.JPG)

    •	Select the “Available” Tab and using the “Filter” select all the below mentioned plugins and 
    click “Install without restart” button
                            Build Pipeline Plugin
                            Build Grpah View Plugin
                            Node and Label Parameter Plugin
                            Docker Plugin
                            Docker Commons Plugin
                            Docker Build Step Plugin
                            Copy Artifact Plugin
                            Groovy
                            HTML Publisher Plugin
![8](https://cloud.githubusercontent.com/assets/20100300/17998795/b9cad0da-6b3b-11e6-8075-7cb0f49c17f7.JPG)

    •	Once all plugins installed , will be changed the status from “Pending” to “Success”
![9](https://cloud.githubusercontent.com/assets/20100300/17998796/b9cdb1e2-6b3b-11e6-9869-241e6a853c1c.JPG)    

    •	Go to “Manage Jenkins” and click the “Manage Nodes” link to create slave jnlp agents.
![10](https://cloud.githubusercontent.com/assets/20100300/17998797/b9d00ca8-6b3b-11e6-93c4-9f3a78801f64.JPG)    

    •	In order to create slave node, click the “New Node” link
![11](https://cloud.githubusercontent.com/assets/20100300/17998798/b9d1d826-6b3b-11e6-9d59-d9dcdb5757a9.JPG)    

    •	Name the node as “agent1”, select the ‘Permanent Agent” radio button and click “OK” button.
![12](https://cloud.githubusercontent.com/assets/20100300/17998799/b9e22122-6b3b-11e6-9607-5aba46671df7.JPG)    

    •	Mention the Remote root directory “/var/jenkins_home” and click the “Save” button.
![13](https://cloud.githubusercontent.com/assets/20100300/17998800/b9f8d9f8-6b3b-11e6-9b7e-12a7d1e06eff.JPG)    

    •	Repeat the above 3 steps for created nodes – “agent2” and “agent3”
    •	Once 3 nodes created, will be seen the agents as below under “Manage Nodes” link.
![14](https://cloud.githubusercontent.com/assets/20100300/17998802/b9ff59d6-6b3b-11e6-8272-156fe4615cf7.JPG)    

    •	Now, select the each agent, copy the secret key which will be used for creation of Jenkins 
    slave Jnlp agents.
![15](https://cloud.githubusercontent.com/assets/20100300/17998804/ba01eff2-6b3b-11e6-9e31-bff1feb86887.JPG)    

    •	Download the apache maven from - https://maven.apache.org/download.cgi 
    •	Copy the maven folder to ““/c/users/Administrator/docker-workspace/Jenkins/Master
    /jenkins_home” directory
![16](https://cloud.githubusercontent.com/assets/20100300/17998801/b9fe07a2-6b3b-11e6-9070-513ae9e9d903.JPG)

    •	Go to “Manage Jenkins” and click the “Global Tool Configuration” link to set the maven path
![17](https://cloud.githubusercontent.com/assets/20100300/17998803/b9ffbe9e-6b3b-11e6-850c-4d28cf6877c6.JPG)

    •	Select ‘Add Maven”, un-check the “Install automatically, specify the “Name” as 
    “'Maven_3.3.9”, “MAVEN_HOME” as /var/Jenkins_home/apache-maven-3.3.9” and click the “Save” button.
![18](https://cloud.githubusercontent.com/assets/20100300/17998805/ba150b96-6b3b-11e6-978c-afcd4937e112.JPG)

## Install Private Registry with nginx enabled (for proxy):
    
    •	Create a directory with name “registry” under “docker-workspace” directory by executing the 
    command “mkdir registry”
    •	Go to directory – “registry” , create two sub directories – “nginx” and “data”
    •	In “registry” folder, create a file with name “docker-compose.yml”, fill with the below content
                            nginx:
                                image: "nginx:latest"
                                ports:
                                    - 5043:443
                                links:
                                    - registry:registry
                                volumes:
                                    - ./nginx/:/etc/nginx/conf.d
                            registry:
                                    image: registry:2
                                ports:
                                    - 5000:5000
                                environment:
                                    REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
                                volumes:
                                    - ./data:/data
    •	Go to the “nginx” folder, create a file with name “registry.conf”, fill with the below content
                            upstream docker-registry {
                                server registry:5000;
                            }
                            server {
                                listen 443;
                                server_name myregistrydomain.com;
                                # SSL
                                # ssl on;
                                # ssl_certificate /etc/nginx/conf.d/domain.crt;
                                # ssl_certificate_key /etc/nginx/conf.d/domain.key;
                                # disable any limits to avoid HTTP 413 for large image uploads
                                client_max_body_size 0;
                                # required to avoid HTTP 411: see Issue #1486 
                                (https://github.com/docker/docker/issues/1486)
                                chunked_transfer_encoding on;
                                location /v2/ {
                                    # Do not allow connections from docker 1.5 and earlier
                                    # docker pre-1.6.0 did not properly set the user agent on ping,
                                    catch "Go *" user agents
                                    if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
                                        return 404;
                                    }
                                    # To add basic authentication to v2 use auth_basic setting plus add_header
                                    # auth_basic "registry.localhost";
                                    # auth_basic_user_file /etc/nginx/conf.d/registry.password;
                                    # add_header 'Docker-Distribution-Api-Version' 'registry/2.0' always;
                                proxy_pass                          http://docker-registry;
                            proxy_set_header  Host              $http_host;   # required for docker client's sake
                                proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
                                proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
                                proxy_set_header  X-Forwarded-Proto $scheme;
                                proxy_read_timeout                  900;
                                }
                            }
    •	Go to “registry” folder , execute the below command to run the private registry
            o	docker-compose up
 
            o	In order to confirm whether the registry related process created or not , execute the
            command – “docker ps –a” which should be in “Up” status
