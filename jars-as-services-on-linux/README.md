# Running Java application as Linux service with systemd

  >*Reference link - <https://medium.com/@ameyadhamnaskar/running-java-application-as-a-service-on-centos-599609d0c641>*.

  I assume you have your java code with all its dependencies in a jar file.This means you should right now be able to run your code using:  
  `(sudo) java -jar Name_Of_File.jar`  

  Ubuntu has a built-in mechanism to create custom services, enabling them to get started at system boot time and start/stop them as a service.  

## Create bash script to run your jar :  

  1. Create a vi /nano file under **/usr/local/bin/** by running the following command  
  eg. **`sudo vi /usr/local/bin/Service_Name.sh`**  
  This will open a file Service_Name.sh.

  1. Paste the following Code in your Service_Name.sh.
  Modify the **SERVICE_NAME(Name of your Service), PATH_TO_JAR(Absolute Path to you jar File)**, and choose a **PID_PATH_NAME**(just replace Service_Name to your Service_Name keeping -pid at end ) for the file you are going to use to store your service ID.Only Changes needed are to the first 3 variables.  

      ```bash
      #!/bin/sh 
      SERVICE_NAME=Your_Service_Name 
      PATH_TO_JAR=/usr/Name_of_User/MyJavaApplication.jar 
      PID_PATH_NAME=/tmp/Service_Name-pid 
      case $1 in 
      start)
            echo "Starting $SERVICE_NAME ..."
        if [ ! -f $PID_PATH_NAME ]; then 
            nohup java -jar $PATH_TO_JAR /tmp 2>> /dev/null >>/dev/null &      
                        echo $! > $PID_PATH_NAME  
            echo "$SERVICE_NAME started ..."         
        else 
            echo "$SERVICE_NAME is already running ..."
        fi
      ;;
      stop)
        if [ -f $PID_PATH_NAME ]; then
              PID=$(cat $PID_PATH_NAME);
              echo "$SERVICE_NAME stoping ..." 
              kill $PID;         
              echo "$SERVICE_NAME stopped ..." 
              rm $PID_PATH_NAME       
        else          
              echo "$SERVICE_NAME is not running ..."   
        fi    
      ;;    
      restart)  
        if [ -f $PID_PATH_NAME ]; then 
            PID=$(cat $PID_PATH_NAME);    
            echo "$SERVICE_NAME stopping ..."; 
            kill $PID;           
            echo "$SERVICE_NAME stopped ...";  
            rm $PID_PATH_NAME     
            echo "$SERVICE_NAME starting ..."  
            nohup java -jar $PATH_TO_JAR /tmp 2>> /dev/null >> /dev/null &            
            echo $! > $PID_PATH_NAME  
            echo "$SERVICE_NAME started ..."    
        else           
            echo "$SERVICE_NAME is not running ..."    
          fi     ;;
      esac
      ```
  1. Write and quit the above file and give execution permisions :  
  ex. **`sudo chmod +x /usr/local/bin/Service_Name.sh`**  

## Test if Java Application is working with the following commands:

  ```bash
  /usr/local/bin/./Service_Name.sh start
  /usr/local/bin/./Service_Name.sh stop
  /usr/local/bin/./Service_Name.sh restart
  ```
## Creating a Service:  
  1. Create a file under **/etc/systemd/system/** with nano or vi and paste the example script below.  
  eg. **`sudo vi /etc/systemd/system/Service_Name.service`**  

  1. Insert the following code in Service_Name:  
      ```bash
      [Unit]
        Description = Java Service
        After network.target = Service_Name.service
      [Service]
        Type = forking
        Restart=always
        RestartSec=1
        SuccessExitStatus=143 
        ExecStart = /usr/local/bin/Service_Name.sh start
        ExecStop = /usr/local/bin/Service_Name.sh stop
        ExecReload = /usr/local/bin/Service_Name.sh reload
      [Install]
        WantedBy=multi-user.target
      ```
  1. Write and quit this file.  
  
  Your Service is all set up. To Test your Java Application as a Service use the following commands to enable/start/stop/:  
  ```bash
  sudo systemctl daemon-reload
  sudo systemctl enable Service_Name
  sudo systemctl start  Service_Name
  sudo systemctl stop   Service_Name
  ```
  This should get your Java Application up and running as a System Service.  
