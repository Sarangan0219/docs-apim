# Getting started with API Traffic Tool

APIM Traffic Tool allows you to simulate real-world traffic on WSO2 API Manager. Follow the steps below to download, configure, and run the tool.

## Download the Traffic Tool

1. Fork the following GitHub repository.

     ```https://github.com/wso2/product-apim-tooling```

2. Navigate to the place where you want to clone the repo and clone the forked repository. 

     ```git clone https://github.com/[git-username]/product-apim-tooling```
     
## Prerequisites

 1. Install <a href="http://java.sun.com/javase/downloads/index.jsp">Oracle Java SE Development Kit (JDK)</a> version JDK 11 or 1.8 and set the <code>JAVA_HOME</code> environment variable. For more information on setting the <code>JAVA_HOME</code> environment variable for different operating systems, see <a href="{{base_path}}/install-and-setup/install/installing-the-product/installing-the-binary/installing-on-linux-or-os-x/">Setup and Install.</a>
 
 2. Download <a href="https://wso2.com/api-management/">WSO2 API Manager 3.2.0.</a>
 
 3. Download and install <a href="https://www.python.org/downloads/">Python 3.6</a> or higher. 
 
 4. Install <a href="https://pypi.org/project/pip/">pip version 3</a> if not already installed.
 
 5. Install required python packages by running the following command in the project home directory.
      ```pip install -r requirement.txt```
 
 6. Download and install Apache [JMeter 5.1.1](http://jmeter.apache.org/download_jmeter.cgi) or higher.
  
    !!! note
        JMeter will only be used by the attack tool. If you're only using the traffic-tool script, no need to install JMeter

 7. Add the following two packages to the <JMETER_HOME>/lib folder.
    - [Apache ivy jar file](https://ant.apache.org/ivy/download.cgi)
    - Attack tool helper package (can be found from `<TOOL_HOME>/resources/add-on/attack-tool-helpers.jar`)
    
    !!! note
        Skip this step if you're only using the traffic-tool.
            
 8. Download an [IP Geolocation Database](https://www.ipinfodb.com/free-database) and add it to `<TOOL_HOME>/resources/libraries/` directory. Include the name of the added file in `<TOOL_HOME>/config/user-settings.yaml` file in front of ip_database under resources (include the name with the CSV file extension). Make sure the column order of the database schema matches the column order in the user-settings.yaml file. The above link contains two types of datasets "Free IP Geo-location Databases" and "Free IP Proxy Databases". Please download an **IP Geo-location dataset**.
 
 9. By default, access tokens get expired after 60 minutes time interval. If you are planning to simulate traffic for more than 1-hour duration, follow the steps in [configuring token expiration time]({{base_path}}/learn/api-security/oauth2/token-expiration/#configuring-the-token-expiration-time).
      Set the value of token_validity_period as appropriate in the `<TOOL_HOME>/config/traffic-tool.yaml` file. 
     
    !!! note
        It is recommended to set user access token validity period to at least 36000 and token_validity_period to -1 for testing environments (more on access tokens).


## Configuring the Tool

Some of the default configurations for the WSO2 API Manager and default scenario are given in all the config files. If you are running WSO2 API Manager on different configurations or using the tool for a custom scenario, you can change the tool configurations as stated below. All configuration files are in the `<TOOL_HOME>/config` folder. 

1. Add your JMeter path in the `<TOOL_HOME>/config/user-settings.yaml` file under path_variables.

2. Enter the correct API Manager version, endpoints, protocol type, host IP, and ports of WSO2 API Manager in the `<TOOL_HOME>/config/apim.yaml file` (Default ports and details can be found at https://apim.docs.wso2.com/en/latest/install-and-setup/setup/reference/default-product-ports/).

3. Add details of each Application (name, description, and API subscriptions) under the apps section in the `<TOOL_HOME>/config/apim.yaml` file. Provide api_subscriptions as comma (,) separated values.

4. Add details of each API (name, description, tags (specify in a list), context, version, resources) under APIs section in `<TOOL_HOME>/config/api_details.yaml` file.

5. Configure the `<TOOL_HOME>/config/traffic-tool.yaml` file as stated below (Configurations for the traffic script).

    * Modify protocol, IP address, and port of the host under the api_host section.
    * Specify the number of users to be created in the no_of_users field under the tool_config section.

6. Configure the `<TOOL_HOME>/config/attack-tool.yaml` file as stated below (Configurations for the attack script).

    * Modify protocol, IP address, and port of the host under the api_host section.
    * Set the attack duration in seconds using attack_duration. (Note: For DOS and DDOS attacks, attack duration is defined per API)
 
!!! note  
    It is recommended to add `setenv.sh` script in the `<TOOL_HOME>/resources/add-on` directory to the <`JMETER_HOME>/bin` directory to increase the heap size of the JMeter if you are simulating DOS or DDOS attacks with heavy concurrency.
   
7. Add swagger definitions for all APIs as separate json files in the `<TOOL_HOME>/data/swagger/` directory. Filename should be equal to <api_name>.json in lower case letters.

!!! note  
    This is an optional step and swagger files will be generated by the tool if not provided.

## Running the Traffic Tool

To use the traffic tool run the following command with the desired argument in a command line inside the `<TOOL_HOME>/bin` folder. To list down available options and their command line arguments, just run the command with the flag `-h`.

```
./traffic-tool.sh -h 
```

You can see a response as below where the options will be mapped to an integer or string argument.

```
Traffic Tool Options
1: Generate data for example scenario
2: Create scenario in APIM
3: Generate access tokens
4: Generate traffic data (without invoking)
5: Simulate traffic
all: Setup scenario and simulate traffic
stop: Stop traffic tool
clean: Cleanup scenario data in API Manager
```

#### 1. Generate Random User Details (Without Application Distribution)

Run the below command to generate a set of random user details. User data will be written to `<TOOL_HOME>/lib/traffic-tool/data/scenario/user_details.yaml` file (applications section of each user will be null).

   -   **Command**
       
       ```
       ./traffic-tool.sh 1
       ```
       On a successful execution you can see a response as below.

   -   **Response**

       ``` bash tab="Response Format"
       <python_path>
       [INFO] <timestamp>: User generation successful!
       [INFO] <timestamp>: User app pattern generation successful!
       ```

       ``` bash tab="Example Response"
       /usr/bin/python3
       [INFO] 2020-10-26 23:57:27.730459: User generation successful!
       [INFO] 2020-10-26 23:57:27.730627: User app pattern generation successful!
       ```
               
#### 2. Create Scenario in WSO2 API Manager

Run the below command to create APIs & applications, subscribe them and signup a set of users in WSO2 API Manager according to a given scenario.
   
   -   **Command**
   
       ```
       ./traffic-tool.sh 2
       ```
       Listed APIs and applications in the config files will be created.
       On a successful execution you can see a response as below.
      
   -   **Response**

       ``` bash tab="Response Format"
       Creating summariser <summary>
       Created the tree successfully using /home/sarangan/Work/Repos/product-apim-tooling/apim-traffic-tool/bin/../lib/traffic-tool/src/jmeter/create_api_scenario.jmx
       Starting standalone test @ <timestamp>
       Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445
       summary +     34 in 00:00:22 =    1.5/s Avg:     1 Min:     0 Max:    29 Err:    34 (100.00%) Active: 1 Started: 4 Finished: 3
       summary +     15 in 00:00:30 =    0.5/s Avg:     1 Min:     0 Max:     2 Err:    15 (100.00%) Active: 1 Started: 4 Finished: 3
       summary =     49 in 00:00:52 =    0.9/s Avg:     1 Min:     0 Max:    29 Err:    49 (100.00%)
       summary +    115 in 00:00:10 =   11.2/s Avg:     0 Min:     0 Max:     1 Err:   115 (100.00%) Active: 0 Started: 6 Finished: 6
       summary =    164 in 00:01:02 =    2.6/s Avg:     0 Min:     0 Max:    29 Err:   164 (100.00%)
       Tidying up ...    @ <timestamp>
       ... end of run
       Script execution completed
       ```

       ``` bash tab="Example Response"
       Creating summariser <summary>
       Created the tree successfully using /home/sarangan/Work/Repos/product-apim-tooling/apim-traffic-tool/bin/../lib/traffic-tool/src/jmeter/create_api_scenario.jmx
       Starting standalone test @ Tue Oct 27 00:02:37 IST 2020 (1603737157766)
       Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445
       summary +     34 in 00:00:22 =    1.5/s Avg:     1 Min:     0 Max:    29 Err:    34 (100.00%) Active: 1 Started: 4 Finished: 3
       summary +     15 in 00:00:30 =    0.5/s Avg:     1 Min:     0 Max:     2 Err:    15 (100.00%) Active: 1 Started: 4 Finished: 3
       summary =     49 in 00:00:52 =    0.9/s Avg:     1 Min:     0 Max:    29 Err:    49 (100.00%)
       summary +    115 in 00:00:10 =   11.2/s Avg:     0 Min:     0 Max:     1 Err:   115 (100.00%) Active: 0 Started: 6 Finished: 6
       summary =    164 in 00:01:02 =    2.6/s Avg:     0 Min:     0 Max:    29 Err:   164 (100.00%)
       Tidying up ...    @ Tue Oct 27 00:03:40 IST 2020 (1603737220435)
       ... end of run
       Script execution completed
       ```

#### 3. Generate Access Tokens

Run the below command to generate a set of access tokens for all the user-application combinations.

   -   **Command**
  
       ```
       ./traffic-tool.sh 3
       ```
       !!! note
           You may have to run this command, if the generated tokens are expired.
      
   -   **Response**

       ``` bash tab="Response Format"
       Creating summariser <summary>
       Created the tree successfully using /home/sarangan/Work/Repos/product-apim-tooling/apim-traffic-tool/bin/../lib/traffic-tool/src/jmeter/generate_token_list.jmx
       Starting standalone test @ Tue Oct 27 00:23:09 IST 2020 (1603738389391)
       Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445
       summary =    150 in 00:00:02 =   71.1/s Avg:     0 Min:     0 Max:    40 Err:   150 (100.00%)
       Tidying up ...    @ Tue Oct 27 00:23:11 IST 2020 (1603738391931)
       ... end of run
       Token generation completed
       /usr/bin/python3
       [INFO] 2020-10-27 00:23:13.899426: User scenario distribution generated successfully
       Script execution completed
       ```

       ``` bash tab="Example Response"
       Creating summariser <summary>
       Created the tree successfully using /home/sarangan/Work/Repos/product-apim-tooling/apim-traffic-tool/bin/../lib/traffic-tool/src/jmeter/generate_token_list.jmx
       Starting standalone test @ Tue Oct 27 00:23:09 IST 2020 (1603738389391)
       Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445
       summary =    150 in 00:00:02 =   71.1/s Avg:     0 Min:     0 Max:    40 Err:   150 (100.00%)
       Tidying up ...    @ Tue Oct 27 00:23:11 IST 2020 (1603738391931)
       ... end of run
       Token generation completed
       /usr/bin/python3
       [INFO] 2020-10-27 00:23:13.899426: User scenario distribution generated successfully
       Script execution completed
       ```      
      
#### 4. Generate the Traffic Dataset without Invoking

Run the below command to generate an API invoking traffic without actually invoking APIs. 

   -   **Command**
  
       ```
       ./traffic-tool.sh 4
       ```
       You will be prompted for a filename. Enter the filename without a file extension(without .txt, .csv, etc) and the output dataset will be saved in `dataset/generated-traffic/filename.csv` directory.
      
   -   **Response**

       ``` bash tab="Response Format"
       <python_path>
       Data generation script started
       ```

       ``` bash tab="Example Response"
       /usr/bin/python3
       Data generation script started
       ```      

#### 5. Simulate a Traffic on API Manager

Run the below command to simulate an API invoking traffic on WSO2 API Manager. 

   -   **Command**
  
       ```
       ./traffic-tool.sh 5
       ```
       You will be prompted for a filename and the script run time. Enter the filename without a file extension(without .txt, .csv, etc) and the output/ dataset will be saved in `dataset/traffic/filename.csv` directory.(Enter the time in minutes when prompted).
      
   -   **Response**
       ```
       Enter filename (without file extension):
       output
       Enter script execution time in minutes:
       5
       ```
       Traffic will be executed throughout the given time. On a successful execution you can see a response as below.
       
       ``` bash tab="Response Format"
       <python_path>
       Traffic tool started. Wait <min> minutes to complete the script
       ```

       ``` bash tab="Example Response"
       /usr/bin/python3
       Traffic tool started. Wait 5 minutes to complete the script
       ```     
#### 6. Stop the Traffic Tool

Run the below command to stop the API invoking or data generation while the script is running.

   -   **Command**
   
       ```
       ./traffic-tool.sh stop
       ```
       On a successful execution you can see a response as below.

   -   **Response**

       ``` bash tab="Response example"
       Traffic Tool Stopped Successfully
       ```

## Running the Attack Tool

To use the attack tool run the following command with the desired argument in a command line inside the `<TOOL_HOME>/bin` folder. 
To list down available options and their command line arguments, just run the command with the flag `-h`. 

````` 
./attack-tool.sh -h
````` 

You can see a response as below where the attack types will be mapped to an integer or string argument.

```
Attack Tool Options
1: DOS attack
2: DDOS attack
3: Abnormal token usage attack
4: Extreme delete attack
5: Stolen token attack
stop: Stop running attack
```
Execute the relevant attacks with the corresponding integer argument.

-   **Command**

       ``` bash tab="Command Format"
       ./attack-tool.sh <integer>
       ```

       ``` bash tab="Example"
       ./attack-tool.sh 1
       ```      