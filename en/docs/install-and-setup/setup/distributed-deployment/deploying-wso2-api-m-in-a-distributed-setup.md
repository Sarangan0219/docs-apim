# Deploying WSO2 API-M in a Distributed Setup

Follow the instructions below to deploy WSO2 API Manager in a distributed environment with its five main components namely Key Manager, Gateway, Publisher, Developer Portal and Traffic Manager.

### Step 1 - Install and configure WSO2 API-M

1.  Download the [WSO2 API Manager](http://wso2.com/products/api-manager/) zip archive into each of the five servers in the cluster for the distributed deployment.
2.  Unzip the WSO2 API Manager zipped archive, and rename each of those directories in the five servers as Key Manager, Gateway, Publisher, Developer Portal, and Traffic Manager.
    Each of these unzipped directories are referred to as `<API-M_HOME>` in this document.
3.  Optimize the distribution for the required profile. 
    
    When a node starts, it starts all the components and features bundled with it. If you are concerned about resource utilization, you can run the product on a specific profile, so that apart from the common features, only the components and features that are required for that particular profile start up.
      
    !!! note
          You can either run the profile optimizer before starting the server or while starting the server. 
          If you are running the optimizer while starting the server make sure optimizer is run with the ```--skipConfigOptimization``` option to preserve the manually applied configurations in the deployment.toml file.
      
    
     For more information on using profile optimizer support, see [Product Profiles]({{base_path}}/install-and-setup/setup/distributed-deployment/product-profiles/).
    
4.  Replace the default certificates (where `CN=localhost`) in each of the five servers, with new certificates generated with proper common name (CN) values to ensure that hostname mismatch issues in the certificates will not occur.
    
    You should use the same primary keystore for all the API Manager instances here in order to decrypt the registry resources. For more information, see [Configuring Primary Keystores]({{base_path}}/administer/product-security/configuring-keystores/configuring-keystores-in-wso2-api-manager/#configuring-the-primary-keystore).
    
    !!! Tip
           When creating the keystore, always use a longer validity period so that it will avoid the need of migration on the registry data when shifting to a new keystore.
    

### Step 2 - Install and configure the databases

You can create the required databases for the API-M deployment in a separate server and point to the databases from the respective nodes. For information on configuring the databases, see [Installing and Configuring the Databases]({{base_path}}/install-and-setup/setting-up-databases/overview/).

### Step 3 - Configure your deployment with production hardening

Ensure that you have taken into account the respective security hardening factors (e.g., changing and encrypting the default passwords, configuring JVM security, etc.) before deploying WSO2 API-M. For more information, see [Production Deployment Guidelines]({{base_path}}/install-and-setup/deploying-wso2-api-manager/production-deployment-guidelines/#common-guidelines-and-checklist).

### Step 4 - Create and import SSL certificates

Create a SSL certificate for each of the WSO2 API-M nodes (i.e. Publisher, Developer Portal, Key Manager, Gateway, and Traffic Manager) and import them to the keyStore and the trustStore. For more information, see [Creating SSL Certificates]({{base_path}}/administer/product-security/configuring-keystores/keystore-basics/creating-new-keystores/).

When maintaining high availability (HA) in the WSO2 API-M distributed set up, you need to create and import a SSL certificate for each of the WSO2 API-M HA nodes.


### Step 5 - Configure API-M Analytics

If you wish to view reports, statistics, and graphs related to the APIs deployed in the Developer Portal, you need to configure API-M Analytics. Follow the [standard setup]({{base_path}}/learn/analytics/configuring-apim-analytics/#standard-setup) to configure API-M Analytics in a production setup, and follow the [quick setup]({{base_path}}/learn/analytics/configuring-apim-analytics/#quick-setup) to configure API-M Analytics in a development setup.

### Step 6 - Configure the connections among the components and start the servers

You will now configure the inter-component relationships of the distributed setup by modifying their `<API-M_HOME>/repository/conf/deployment.toml` files. Once the required configuration is done in each component, it is recommended to start the them in the following order:Key Manager, Traffic Manager, Publisher, Developer Portal, and Gateway.

!!! note
    In a clustered environment, you use session affinity (sticky sessions) to ensure that requests from the same client always get routed to the same server.
 
    When an API is published from the API Publisher, a file with its synapse configuration is puhsed to the API Gateway.Therefore in order to ensure that Basic Auth authentication for the respective admin service call and subsequent synapse artifact deployment requests are sent to the same Gateway node, we need to enable sticky sessions for the servlet transport ports ( i.e 9443 if no port offsert is configured ) in the load balancer that is fronting the Gateway nodes.
    
    Similarly when a throttle policy is created from the Admin dashboard (Publisher Node), a Siddhi execution plan is created and deployed in Traffic Manager via an admin service call and therefore sticky sessions needs to be enabled for the servlet transport ports ( i.e 9443 if no port offsert is configured ) in the load balancer that is fronting the Traffic Manager nodes.
    
    Key validation requests sent from the Gateway node to the Key Manager nodes also happen via an admin service call, therefore sticky sessions needs to be enabled for the servlet transport ports ( i.e 9443 if no port offsert is configured ) in the load balancer that is fronting the Key Manager nodes.
    
-   [Step 6.1 - Configure and start the Key Manager](#step-62-configure-and-start-the-key-manager)
-   [Step 6.2 - Configure and start the Traffic Manager](#step-63-configure-and-start-the-traffic-manager)
-   [Step 6.3 - Configure and start the API Publisher](#step-64-configure-and-start-the-api-publisher)
-   [Step 6.4 - Configure and start the Developer Portal](#step-65-configure-and-start-the-developer-portal)
-   [Step 6.5 - Configure and start the Gateway](#step-66-configure-and-start-the-gateway)

#### Step 6.1 - Configure and start the Key Manager

This section involves setting up the Key Manager node and enabling it to work with the other components in a distributed deployment.

[![Key Manager Connections]({{base_path}}/assets/img/setup-and-install/key-manager-connections.png)]({{base_path}}/assets/img/setup-and-install/key-manager-connections.png)


!!! warning
    **Skip** this step if you are using **WSO2 Identity Server as the Key Manager** and follow the instructions mentioned in [Configuring WSO2 Identity Server as a Key Manager]({{base_path}}/install-and-setup/deploying-wso2-api-manager/distributed-deployment/configuring-wso2-identity-server-as-a-key-manager/) to configure and start the Key Manager.
1.  Open the `<API-M_HOME>/repository/conf/deployment.toml` file in the Key Manager node and change `[apim.throttling]` section to point to the Traffic Manager nodes.
 
 
     ``` toml tab="Traffic Manager with HA"
     [apim.throttling]
     throttle_decision_endpoints = ["tcp://Traffic-Manager-1-host:5672","tcp://Traffic-Manager-2-host:5672"]
     
     [[apim.throttling.url_group]]
     traffic_manager_urls = ["tcp://Traffic-Manager-1-host:9611"]
     traffic_manager_auth_urls = ["ssl://Traffic-Manager-1-host:9711"]
     
     [[apim.throttling.url_group]]
     traffic_manager_urls = ["tcp://Traffic-Manager-2-host:9611"]
     traffic_manager_auth_urls = ["ssl://Traffic-Manager-2-host:9711"]
     ```

     ``` toml tab="Single Traffic Manager"
     [apim.throttling]
     throttle_decision_endpoints = ["tcp://Traffic-Manager-host:5672"]
          
     [[apim.throttling.url_group]]
     traffic_manager_urls = ["tcp://Traffic-Manager-host:9611"]
     traffic_manager_auth_urls = ["ssl://Traffic-Manager-host:9711"]
     ```
   
2.  If you wish to encrypt the Auth Keys (access tokens, client secrets, and authorization codes), see [Encrypting OAuth Keys]({{base_path}}/learn/api-security/oauth2/encrypting-oauth2-tokens/).


3.  If you need to configure High Availability (HA) for the Key Manager, use a copy of the active instance configured above as the second Key Manager active instance and configure a load balancer fronting the two Key Manager instances.
    
    For information on configuring the load balancer, see [Configuring the Proxy Server and the Load Balancer]({{base_path}}/install-and-setup/deploying-wso2-api-manager/configuring-the-proxy-server-and-the-load-balancer/).

4.  Start the WSO2 API-M Key Manager node(s) by running the below command in the command prompt. For more information on starting a WSO2 server, see [Starting the server]({{base_path}}/install-and-setup/installation-guide/running-the-product/#starting-the-server).

    ``` java tab="Linux/Mac OS"
    cd <API-M_HOME>/bin/
    sh wso2server.sh -Dprofile=api-key-manager
    ```

    ``` java tab="Windows"
    cd <API-M_HOME>\bin\
    wso2server.bat --run -Dprofile=api-key-manager
    ```

??? info "Click here to view sample configuration for the Key Manager"
    ``` toml
    [server]
    server_role = "api-key-manager"
    hostname = "km.wso2.com"
    node_ip = "127.0.0.1"
    offset=0
    
    [user_store]
    type = "database"
    
    [super_admin]
    username = "admin"
    password = "admin"
    create_admin_account = true
    
    [database.apim_db]
    type = "mysql"
    hostname = "db.wso2.com"
    name = "apim_db"
    port = "3306"
    username = "root"
    password = "root"
    
    [database.shared_db]
    type = "mysql"
    hostname = "db.wso2.com"
    name = "shared_db"
    port = "3306"
    username = "root"
    password = "root"
    
    [keystore.tls]
    file_name =  "wso2carbon.jks"
    type =  "JKS"
    password =  "wso2carbon"
    alias =  "wso2carbon"
    key_password =  "wso2carbon"
    
    [truststore]
    file_name = "client-truststore.jks"
    type = "JKS"
    password = "wso2carbon"
    
    [apim.throttling]
    throttle_decision_endpoints = ["tcp://tm.wso2.com:5675"]
    
    [[apim.throttling.url_group]]
    traffic_manager_urls=["tcp://tm.wso2.com:9614"]
    traffic_manager_auth_urls=["ssl://tm.wso2.com:9714"]

    ```

#### Step 6.2 - Configure and start the Traffic Manager

This section involves setting up the Traffic Manager node(s) and enabling it to work with the other components in a distributed deployment.

[![Traffic Manager Connections]({{base_path}}/assets/img/setup-and-install/traffic-manager-connections.png)]({{base_path}}/assets/img/setup-and-install/traffic-manager-connections.png)

1.  If you need to configure High Availability (HA) for the Traffic Manager, use a copy of the existing Traffic Manager instance as the second Traffic Manager active instance and configure a load balancer fronting the two Traffic Manager instances.

2.  Mount the `<API-M_HOME>/repository/deployment/server/executionplans` directory of all the Traffic Manager nodes to a shared file system as a content synchronization mechanism in-order to share the throttling policies between Traffic Manager nodes.
  
3.  Start the WSO2 API-M Traffic Manager node(s) by running the below command in the command prompt. For more information on starting a WSO2 server, see [Starting the server]({{base_path}}/install-and-setup/installation-guide/running-the-product/#starting-the-server).

    ``` java tab="Linux/Mac OS"
    cd <API-M_HOME>/bin/
    sh wso2server.sh -Dprofile=traffic-manager
    ```

    ``` java tab="Windows"
    cd <API-M_HOME>\bin\
    wso2server.bat --run -Dprofile=traffic-manager
    ```


    ??? info "Click here to view sample configuration for the Traffic Manager"
        ``` toml
        [server]
        hostname = "tm.wso2.com"
        node_ip = "127.0.0.1"
        server_role = "traffic-manager"
        offset=3
        
        [user_store]
        type = "database"
        
        [super_admin]
        username = "admin"
        password = "admin"
        create_admin_account = true
        
        [database.shared_db]
        type = "h2"
        url = "jdbc:h2:./repository/database/WSO2SHARED_DB;DB_CLOSE_ON_EXIT=FALSE"
        username = "wso2carbon"
        password = "wso2carbon"
        
        [keystore.tls]
        file_name =  "wso2carbon.jks"
        type =  "JKS"
        password =  "wso2carbon"
        alias =  "wso2carbon"
        key_password =  "wso2carbon"
        
        [truststore]
        file_name = "client-truststore.jks"
        type = "JKS"
        password = "wso2carbon"
        ```



#### Step 6.3 - Configure and start the API Publisher

This section involves setting up the API Publisher node and enabling it to work with the other components in the distributed deployment.

[![Publisher Connections]({{base_path}}/assets/img/setup-and-install/publisher-connections.png)]({{base_path}}/assets/img/setup-and-install/publisher-connections.png)

1.  Open the `<API-M_HOME>/repository/conf/deployment.toml` file in the API Publisher node and make the following changes.

    1.  Configure the Publisher with the Traffic Manager.
        This configuration enables publishing of throttling policies, custom templates, and block conditions to the Traffic Manager node.

        ``` toml tab="Traffic Manager with HA"
        [apim.throttling]
        service_url = "https://[Traffic-Manager-LB-Host]/services/"
        throttle_decision_endpoints = ["tcp://Traffic-Manager-1-host:5672","tcp://Traffic-Manager-2-host:5672"]
        
        [[apim.throttling.url_group]]
        traffic_manager_urls = ["tcp://Traffic-Manager-1-host:9611"]
        traffic_manager_auth_urls = ["ssl://Traffic-Manager-1-host:9711"]
        
        [[apim.throttling.url_group]]
        traffic_manager_urls = ["tcp://Traffic-Manager-2-host:9611"]
        traffic_manager_auth_urls = ["ssl://Traffic-Manager-2-host:9711"]
        ```
        
        ``` toml tab="Single Traffic Manager"
        [apim.throttling]
        service_url = "https://Traffic-Manager-host:${mgt.transport.https.port}/services/"
        throttle_decision_endpoints = ["tcp://Traffic-Manager-host:5672"]

        [[apim.throttling.url_group]]
        traffic_manager_urls = ["tcp://Traffic-Manager-host:9611"]
        traffic_manager_auth_urls = ["ssl://Traffic-Manager-host:9711"]
        ```

    2.  Configure the Publisher with the Gateway.
        This configuration enables pushing the synapse artifact file to the gateway when an api is published.

        -   If you are using a single Gateway node, configure Publisher with the Gateway as follows:

            ``` toml
            [[apim.gateway.environment]]
            name = "Production and Sandbox"
            type = "hybrid"
            display_in_api_console = true
            description = "This is a hybrid gateway that handles both production and sandbox token traffic."
            show_as_token_endpoint_url = true
            service_url = "https://[API-Gateway-Host-or-IP]:${mgt.transport.https.port}/services/"
            username= "${admin.username}"
            password= "${admin.password}"
            ws_endpoint = "ws://[API-Gateway-Host-or-IP]:9099"
            wss_endpoint = "wss://[API-Gateway-Host-or-IP]:8099"
            http_endpoint = "http://[API-Gateway-Host-or-IP]:${http.nio.port}"
            https_endpoint = "https://[API-Gateway-Host-or-IP]:${https.nio.port}"
            ```

        -   If you are using multiple Gateway nodes, configure Publisher with the Gateway nodes as follows based on your content synchronization mechanism.

            ``` toml tab="With shared file system"
            [[apim.gateway.environment]]
            name = "Production and Sandbox"
            type = "hybrid"
            display_in_api_console = true
            description = "This is a hybrid gateway that handles both production and sandbox token traffic."
            show_as_token_endpoint_url = true
            service_url = "https://[API-Gateway-LB-Host]/services/"
            username= "${admin.username}"
            password= "${admin.password}"
            ws_endpoint = "ws://[API-Gateway-LB-Host-or-IP]:9099"
            wss_endpoint = "wss://[API-Gateway-LB-Host-or-IP]:8099"
            http_endpoint = "http://[API-Gateway-LB-Host]"
            https_endpoint = "https://[API-Gateway-LB-Host]"
            ```

            ``` tab="With rsync"
            [[apim.gateway.environment]]
            name = "Production and Sandbox"
            type = "hybrid"
            display_in_api_console = true
            description = "This is a hybrid gateway that handles both production and sandbox token traffic."
            show_as_token_endpoint_url = true
            service_url = "https://[API-Gateway-LB-Host]/services/"
            username= "${admin.username}"
            password= "${admin.password}"
            ws_endpoint = "ws://[API-Gateway-LB-Host]:9099"
            wss_endpoint = "wss://[API-Gateway-LB-Host]:8099"
            http_endpoint = "http://[API-Gateway-LB-Host]"
            https_endpoint = "https://[API-Gateway-LB-Host]"
            ```

    3.  Configure the Developer Portal URL to appear in the Publisher UI.

        ``` toml tab="Developer Portal with HA"
        [apim.devportal]
        url = "https://[devportal-LB-hostname]/devportal"        
        ```
        
        ``` toml tab="Single Developer Portal"
        [apim.devportal]
        url = "https://[devportal-hostname]:${mgt.transport.https.port}/devportal"
        ```

4.  If you need to configure High Availability (HA) for the Api Publisher nodes, use a copy of the active instance configured above as the second active Publisher instance and configure a load balancer fronting the two Publisher instances.
           
    For information on configuring the load balancer, see [Configuring the Proxy Server and the Load Balancer]({{base_path}}/install-and-setup/deploying-wso2-api-manager/configuring-the-proxy-server-and-the-load-balancer/).

5.  Start the WSO2 API-M Publisher node(s) by running the below command in the command prompt.
    For more information on starting a WSO2 server, see [Starting the server]({{base_path}}/install-and-setup/installation-guide/running-the-product/#starting-the-server).

    ``` java tab="Linux/Mac OS"
    cd <API-M_HOME>/bin/
    sh wso2server.sh -Dprofile=api-publisher
    ```
    
    ``` java tab="Windows"
    cd <API-M_HOME>\bin\
    wso2server.bat --run -Dprofile=api-publisher
    ```

??? info "Click here to view sample configuration for the Publisher"
    ``` toml
    [server]
    hostname = "pub.wso2.com"
    node_ip = "127.0.0.1"
    server_role = "api-publisher"
    offset=1
    
    [user_store]
    type = "database"
    
    [super_admin]
    username = "admin"
    password = "admin"
    create_admin_account = true
    
    [database.apim_db]
    type = "mysql"
    hostname = "db.wso2.com"
    name = "apim_db"
    port = "3306"
    username = "root"
    password = "root"
    
    [database.shared_db]
    type = "mysql"
    hostname = "db.wso2.com"
    name = "shared_db"
    port = "3306"
    username = "root"
    password = "root"
    
    [keystore.tls]
    file_name =  "wso2carbon.jks"
    type =  "JKS"
    password =  "wso2carbon"
    alias =  "wso2carbon"
    key_password =  "wso2carbon"
    
    [truststore]
    file_name = "client-truststore.jks"
    type = "JKS"
    password = "wso2carbon"
    
    [[apim.gateway.environment]]
    name= "Production and Sandbox"
    type= "hybrid"
    display_in_api_console= true
    description= "This is a hybrid gateway that handles both production and sandbox token traffic."
    service_url= "https://gw.wso2.com:9447/services/"
    http_endpoint = "http://gw.wso2.com:8284"
    https_endpoint = "https://gw.wso2.com:8247"
    username= "${admin.username}"
    password= "${admin.password}"
    
    [apim.throttling]
    service_url = "https://tm.wso2.com:9446/services/"
    throttle_decision_endpoints = ["tcp://tm.wso2.com:5675"]
    username= "$ref{super_admin.username}"
    password= "$ref{super_admin.password}"
    
    [[apim.throttling.url_group]]
    traffic_manager_urls=["tcp://tm.wso2.com:9614"]
    traffic_manager_auth_urls=["ssl://tm.wso2.com:9714"]
    
    [apim.devportal]
    url = "https://store.wso2.com:9445/devportal"

    ```
#### Step 6.4 - Configure and start the Developer Portal

This section involves setting up the Developer Portal node and enabling it to work with the other components in the distributed deployment.

[![Developer Portal Connections]({{base_path}}/assets/img/setup-and-install/dev-portal-connections.png)]({{base_path}}/assets/img/setup-and-install/dev-portal-connections.png)

1.  Open the `<API-M_HOME>/repository/conf/deployment.toml` file in the Developer Portal node and make the following changes.

    1.  Configure the Developer Portal with the Key Manager.

        
        ``` toml tab="Key Manager with HA"
        [apim.key_manager]
        service_url = "https://[Key-Manager-LB-host]/services/"
        username = "$ref{super_admin.username}"
        password = "$ref{super_admin.password}"
        ```
        
        ``` toml tab="Single Key Manager"
        [apim.key_manager]
        service_url = "https://[Key-Manager-host]:${mgt.transport.https.port}/services/"
        username = "$ref{super_admin.username}"
        password = "$ref{super_admin.password}"
        ```

    2.  Configure the Developer Portal with the Traffic Manager.

        ``` toml tab="Traffic Manager with HA"
        [apim.throttling]
        throttle_decision_endpoints = ["tcp://Traffic-Manager-1-host:5672","tcp://Traffic-Manager-2-host:5672"]
        
        [[apim.throttling.url_group]]
        traffic_manager_urls = ["tcp://Traffic-Manager-1-host:9611"]
        traffic_manager_auth_urls = ["ssl://Traffic-Manager-1-host:9711"]
        
        [[apim.throttling.url_group]]
        traffic_manager_urls = ["tcp://Traffic-Manager-2-host:9611"]
        traffic_manager_auth_urls = ["ssl://Traffic-Manager-2-host:9711"]
        ```
        
        ``` toml tab="Single Traffic Manager"
        [apim.throttling]
        throttle_decision_endpoints = ["tcp://Traffic-Manager-host:5672"]
             
        [[apim.throttling.url_group]]
        traffic_manager_urls = ["tcp://Traffic-Manager-host:9611"]
        traffic_manager_auth_urls = ["ssl://Traffic-Manager-host:9711"]
        ```

    3.  Configure the Token Revoke endpoint to point to Key Manager.

        
        ``` toml tab="Multiple Gateways"
        [apim.oauth_config]
        revoke_endpoint = "https://[API-Key-Manager-LB-Host]/oauth2/revoke"
        ```
        
        ``` toml tab="Single Gateway"
        [apim.oauth_config]
        revoke_endpoint = "https://[API-Key-Manager-host-or-IP]:${mgt.transport.https.port}/oauth2/revoke"
        ```
    
    4.  Configure the Developer Portal with Gateway.
   
           
           ``` toml tab="Gateway with HA"
           [[apim.gateway.environment]]
           name = "Production and Sandbox"
           type = "hybrid"
           display_in_api_console = true
           description = "This is a hybrid gateway that handles both production and sandbox token traffic."
           show_as_token_endpoint_url = true
           ws_endpoint = "ws://[API-Gateway-LB-Host-or-IP]:9099"
           wss_endpoint = "wss://[API-Gateway-LB-Host-or-IP]:8099"
           http_endpoint = "http://[API-Gateway-LB-Host]"
           https_endpoint = "https://[API-Gateway-LB]"
           ```
           
           ``` toml tab="Single Gateway"
           [[apim.gateway.environment]]
           name = "Production and Sandbox"
           type = "hybrid"
           display_in_api_console = true
           description = "This is a hybrid gateway that handles both production and sandbox token traffic."
           show_as_token_endpoint_url = true
           ws_endpoint = "ws://[API-Gateway-host-or-IP]:9099"
           wss_endpoint = "wss://[API-Gateway-host-or-IP]:8099"
           http_endpoint = "http://[API-Gateway-host-or-IP]:${http.nio.port}"
           https_endpoint = "https://[API-Gateway-host-or-IP]:${https.nio.port}"
           ```

3.  If you need to configure High Availability (HA) for the Developer Portal nodes, use a copy of the active instance configured above as the second active Developer Portal instance and configure a load balancer fronting the two Developer Portal instances.
            
    For information on configuring the load balancer, see [Configuring the Proxy Server and the Load Balancer]({{base_path}}/install-and-setup/deploying-wso2-api-manager/configuring-the-proxy-server-and-the-load-balancer/)
        
4.  Start the Developer Portal node(s) by running the below command in the command prompt. 
    For more information on starting a WSO2 server, see [Starting the server]({{base_path}}/install-and-setup/installation-guide/running-the-product/#starting-the-server).

    ``` java tab="Linux/Mac OS"
    cd <API-M_HOME>/bin/
    sh wso2server.sh -Dprofile=api-devportal
    ```
    
    ``` java tab="Windows"
    cd <API-M_HOME>\bin\
    wso2server.bat --run -Dprofile=api-devportal
    ```
    

    ??? info "Click here to view sample configuration for the Developer Portal"
        ``` toml
        [server]
        hostname = "store.wso2.com"
        node_ip = "127.0.0.1"
        server_role="api-store"
        offset=2
        
        [user_store]
        type = "database"
        
        [super_admin]
        username = "admin"
        password = "admin"
        create_admin_account = true
        
        [database.apim_db]
        type = "mysql"
        hostname = "db.wso2.com"
        name = "apim_db"
        port = "3306"
        username = "root"
        password = "root"
        
        [database.shared_db]
        type = "mysql"
        hostname = "db.wso2.com"
        name = "shared_db"
        port = "3306"
        username = "root"
        password = "root"
        
        [keystore.tls]
        file_name =  "wso2carbon.jks"
        type =  "JKS"
        password =  "wso2carbon"
        alias =  "wso2carbon"
        key_password =  "wso2carbon"
        
        [truststore]
        file_name = "client-truststore.jks"
        type = "JKS"
        password = "wso2carbon"
        
        [[apim.gateway.environment]]
        name= "Production and Sandbox"
        type= "hybrid"
        display_in_api_console= true
        description= "This is a hybrid gateway that handles both production and sandbox token traffic."
        ws_endpoint= "ws://gw.wso2.com:9099"
        http_endpoint = "http://gw.wso2.com:8284"
        https_endpoint = "https://gw.wso2.com:8247"
        show_as_token_endpoint_url = true
        
        [apim.key_manager]
        service_url = "https://km.wso2.com:9443/services/"
        username= "$ref{super_admin.username}"
        password= "$ref{super_admin.password}"
        
        [apim.oauth_config]
        revoke_endpoint = "https://km.wso2.com:9443/revoke"
        
        [apim.throttling]
        throttle_decision_endpoints = ["tcp://tm.wso2.com:5675"]
        
        [[apim.throttling.url_group]]
        traffic_manager_urls=["tcp://tm.wso2.com:9614"]
        traffic_manager_auth_urls=["ssl://tm.wso2.com:9714"]
        
        ```

#### Step 6.5 - Configure and start the Gateway

This section involves setting up the Gateway node and enabling it to work with the other components in the distributed deployment.

[![Gateway Connections]({{base_path}}/assets/img/setup-and-install/gateway-connections.png)]({{base_path}}/assets/img/setup-and-install/gateway-connections.png)

1.  If you need to configure High Availability (HA) for the Gateway nodes, follow below steps.

      1.  Open `<API-M_HOME>/repository/conf/deployment.toml` and locate the `<HostName>` tag to add the cluster hostname. 
          
          For an example, if the hostname is `gw.am.wso2.com` :
    
          ``` toml
          [server]
          hostname = "gw.wso2.com"
          ```
          
      2.  Specify the following incoming connection configurations in the same `deployment.toml` file.
      
          ``` toml
          [transport.http]
          properties.port = 9763
          properties.proxyPort = 80
          
          [transport.https]
          properties.port = 9443
          properties.proxyPort = 443
          ```
      
      3.  Open the server's `/etc/hosts` file and map the hostnames to IPs.
      
          ```java tab="Format"
          <GATEWAY-IP> gw.wso2.com
          ```
      
          ``` java tab="Example"
          xxx.xxx.xxx.xx4 gw.wso2.com
          ```

      4.  Mount the `<API-M_HOME>/repository/deployment/server` directory of all the Gateway nodes to the shared file system to share all APIs between the Gateway nodes.
 
        !!! note
              WSO2 recommends using a shared file system as the content synchronization mechanism to synchronize the artifacts among the WSO2 API-M Gateway nodes, because a shared file system does not require a specific node to act as a Gateway Manager, instead all the nodes have the worker manager capabilities.
              For this purpose you can use a common shared file system such as Network File System ( NFS ) or any other shared file system.
              
              **What if I am unable to use a shared file system?**
              
              If you are unable to have a shared file system, you can use remote synchronization (rsync) instead, but note that when using rsync there is the vulnerability of a single point of failure, because rsync needs one node to act as the Gateway Manager as it only provides write permission to one node. For more information, see [Configuring rsync for Deployment Synchronization]({{base_path}}/install-and-setup/setup/distributed-deployment/clustering-gateway-for-ha-using-rsync/).
              
                                                                
2.  Modify the `<API-M_HOME>/repository/conf/deployment.toml` file in the Gateway node to configure the connection to the Key Manager component.


    ``` toml tab="Key Managers with HA"
    [apim.key_manager]
    service_url = "https://[Key-Manager-LB-host]/services/"
    username = "$ref{super_admin.username}"
    password = "$ref{super_admin.password}"                
    ```
    
    ``` toml tab="Single Key Manager"
    [apim.key_manager]
    service_url = "https://[Key-Manager-host]:${mgt.transport.https.port}/services/"
    username = "$ref{super_admin.username}"
    password = "$ref{super_admin.password}"
    ```

4.  If you need to enable JSON Web Token (JWT), you have to enable it in all Gateway and Key Manager components.
    For more information on configuring JWT, see [Generating JSON Web Token]({{base_path}}/learn/api-gateway/passing-end-user-attributes-to-the-backend/passing-enduser-attributes-to-the-backend-using-jwt/).

5.   Modify the `<API-M_HOME>/repository/conf/deployment.toml` file in the Gateway node to communicate with the Traffic Manager node(s).
     
     ``` toml tab="Traffic Manager with HA"
     [[apim.throttling.url_group]]
     traffic_manager_urls = ["tcp://Traffic-Manager-1-host:9611"]
     traffic_manager_auth_urls = ["ssl://Traffic-Manager-1-host:9711"]
     
     [[apim.throttling.url_group]]
     traffic_manager_urls = ["tcp://Traffic-Manager-2-host:9611"]
     traffic_manager_auth_urls = ["ssl://Traffic-Manager-2-host:9711"]
     
     [apim.throttling]
     throttle_decision_endpoints = ["tcp://Traffic-Manager-1-host:5672", "tcp://Traffic-Manager-2-host:5672"]
     ```
     
     ``` toml tab="Single Traffic Manager"
     [[apim.throttling.url_group]]
     traffic_manager_urls = ["tcp://Traffic-Manager-host:9611"]
     traffic_manager_auth_urls = ["ssl://Traffic-Manager-host:9711"]
     
     [apim.throttling]
     throttle_decision_endpoints = ["tcp://Traffic-Manager-host:5672"]
     ```
     
6.  If Gateways are configured for High Availability (HA), use a copy of the active instance configured above as the second active Gateway instance and configure a load balancer fronting the two Gateway instances.
                
    For information on configuring the load balancer, see [Configuring the Proxy Server and the Load Balancer]({{base_path}}/install-and-setup/deploying-wso2-api-manager/configuring-the-proxy-server-and-the-load-balancer/)
        
7.  Start the Gateway node(s) by running the below command in the command prompt. 
    For more information on starting a WSO2 server, see [Starting the server]({{base_path}}/install-and-setup/installation-guide/running-the-product/#starting-the-server).

    ``` java tab="Linux/Mac OS"
    cd <API-M_HOME>/bin/
    sh wso2server.sh -Dprofile=gateway-worker
    ```
    
    ``` java tab="Windows"
    cd <API-M_HOME>\bin\
    wso2server.bat --run -Dprofile=gateway-worker
    ```
    

    ??? info "Click here to view sample configuration for the Gateway"
        ``` toml
        [server]
        hostname = "gw.wso2.com"
        node_ip = "127.0.0.1"
        server_role = "gateway-worker"
        offset=4
        
        [user_store]
        type = "database"
        
        [super_admin]
        username = "admin"
        password = "admin"
        create_admin_account = true
        
        [database.shared_db]
        type = "mysql"
        hostname = "db.wso2.com"
        name = "shared_db"
        port = "3306"
        username = "root"
        password = "root"
        
        [keystore.tls]
        file_name =  "wso2carbon.jks"
        type =  "JKS"
        password =  "wso2carbon"
        alias =  "wso2carbon"
        key_password =  "wso2carbon"
        
        [truststore]
        file_name = "client-truststore.jks"
        type = "JKS"
        password = "wso2carbon"
        
        [apim.key_manager]
        service_url = "https://km.wso2.com:9443/services/"
        
        [apim.throttling]
        throttle_decision_endpoints = ["tcp://tm.wso2.com:5675"]
        
        [[apim.throttling.url_group]]
        traffic_manager_urls=["tcp://tm.wso2.com:9614"]
        traffic_manager_auth_urls=["ssl://tm.wso2.com:9714"]
        
        [apim.cors]
        allow_origins = "*"
        allow_methods = ["GET","PUT","POST","DELETE","PATCH","OPTIONS"]
        allow_headers = ["authorization","Access-Control-Allow-Origin","Content-Type","SOAPAction"]
        allow_credentials = false
        
        
        ```



