# Adavanced Configurations for API Traffic tool

By default, the tool will be running on default configurations and settings. However, you can further configure the tool as below.

## Managing tenants

By default whole procedure happens on the super tenant of the API Manager. However traffic tool allows to change the setup environment. Enter below details under main_tenant section in the `<TOOL_HOME>/config/apim.yaml` file.


- tenant_name: Name of the tenant (Default is super)
- admin_username: Username of the tenant admin
- admin_password: Password of the tenant admin
- admin_b64: Base64 encrypted value of the admin_username:admin_password

Make sure self signup enabled in WSO2 API Manager. By default it will be disabled for all tenants except the super tenant. (Enabling Self Signup)

## Advanced Configurations for the Traffic Script

Change throttling tier, visibility, production and sandbox endpoints of the APIs under api section (APIs are created for these configurations).

Change throttling tier and token validity period of the applications under application section (Applications are created for these configurations).

Change subscription tier of the app api subscriptions.

Configure tool_config section as to the following definitions.
    max_connection_refuse_count: Maximum number of connection refuse count allowed. Traffic tool will stop after the given number of connection refuses.
    no_of_data_points: No of data points or requests to be generated when generating the traffic data without invoking.
    heavy_traffic: If you want to simulate a heavy traffic, set this value as true. Otherwise set it to false.
    'frequency_limits': This section lists request frequency limits for low, medium and high frequent time patterns. These lower and upper bounds will be used when generating a random invoke scenario. This section doesn't apply if you are using the tool for the example scenario.

    It is recommended to set heavy_traffic to false in model training and testing environments to have a better training for attacks.

Append, modify or remove payloads under payloads section to send with POST and DELETE requests when simulating a normal traffic. A random payload will be sent with each POST or DELETE request. If your DELETE endpoint operates with path parameters, leave payloads empty under delete section.

Append, modify or remove user agents under user_agents section. These user agents are sent randomly when simulating a normal traffic.

## Advanced Configurations for the Attack Script (<TOOL_HOME>/config/attack-tool.yaml)

Append, modify or remove user agents from the list under user_agents section.

Configure the concurrency using number_of_processes.

Append, modify or remove payloads from the list under payloads section. These payloads are used in the bodies of POST,PUT and PATCH requests.

Set minimum and maximum request scalars under abnormal_token_usage. These will be used to scale the normal traffic in order to generate attack traffic for simulating abnormal token usage attack.


## Managing Tenants

In the tool users are simulated through python processes. Those processes invoke and sleep according to a given time pattern. These patterns are listed in `<TOOL_HOME>/lib/traffic-tool/data/tool_data/invoke_patterns.yaml` file. You can add, modify or remove invoke patterns under time_patterns section (time intervals as comma separated integers). 

   -   **Time Patterns**

       ``` bash tab="Time Patterns"
       time_patterns:
       pattern1: 0,100,0,2,300,500,1,2,100,400,700,1,0,0,900,30,600,1800,1800,2,400,6,100,0,0,200,100,0,100
       pattern2: 0,1,0,0,2,0,5,0,1,0,10,0,15,0,20,15,1800,0,1,0,0,0,2,1,0,1,0,0,15,0,10,0,20
       pattern3: 0,1,0,0,2,0,5,0,1,0,10,0,15,0,20,15,600,0,1,0,0,0,2,1,600,0,1,0,0,15,0,10,0,20
       ```
     
After adding (or removing) a time pattern, please mention (or remove) it in (or from) the frequency section.

   -   **Frequency**

       ``` bash tab="Frequency"
       frequency:
       low: pattern1
       medium: pattern2, pattern3
       high: pattern8
       ```
       
!!! note 
    You can simulate a more realistic real world access pattern by adding time patterns from real world datasets.


## Creating Custom APIM Scenario

You can make the tool invoking happen to a custom scenario in a few simple steps. First you have to think and design a real world API access pattern which is similar to the example scenario given. Then follow below steps to change scenario data files.

  1. Define APIs and applications Add APIs & applications, and define their subscription patterns as mentioned in "Configuring the Tool" section. Make sure to provide swagger definitions for all mentioned APIs.

  2. Provide a set of applications for each user Generate a set of random user details by running `./traffic-tool.sh 1`. Applications for generated users will be `null` in the `<TOOL_HOME>/lib/traffic-tool/data/scenario/user_details.yaml` file. Specify application/s for each user as comma (,) separated values.

       ``` 
       users:
         - username: jamie1
           password: jamie1
           firstname: Jamie
           ...
           applications: Online Shopping,Taxi
       ```
    
 3. Define the invoke scenario Define the invoke scenario in `<TOOL_HOME>/lib/traffic-tool/data/scenario/invoke_scenario.yaml` file. Each record under invoke_scenario is for a different user type in the scenario table.

        * app_name: Name of the application to be invoked
        * no_of_users: No of users for the considered scenario
        * time_pattern: Invoke pattern for the user type. Time patterns can be added to <TOOL_HOME>/lib/traffic-tool/data/tool_data/invoke_patterns.yaml file as a comma seperated list.
        * api_calls: api name, http method and no of requests from the user type.


        ``` 
        invoke_scenario:
          - app_name: Online Shopping
            no_of_users: 5
            time_pattern: pattern1
            api_calls:
              - api: news
                method: GET
                no_of_requests: 1
        
              - api: places
                method: GET
                no_of_requests: 2
        ```
        
  4. After configuring the tool as to the above steps, run below commands to setup and generate a traffic.
   
     * Create scenario in WSO2 API Manager by running `./traffic-tool.sh 3`
     * Generate access tokens by running `./traffic-tool.sh 4` 
     * Start the traffic tool by running `./traffic-tool.sh start`


