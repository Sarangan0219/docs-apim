# Overview of the WSO2 API Traffic Tool

The tool is consisting of two main components traffic tool and attack tool.  The traffic tool will simulate a real-world traffic pattern while the attack tool will simulate a given attack on WSO2 API Manager. This tool is not just a simple request sender, but it also allows you to set up a user scenario in WSO2 API Manager and generate a set of tokens to invoke APIs.  APIM Traffic Tool can also be used to generate an API invoking traffic dataset and attack dataset for Machine Learning purposes.

## How does APIM Traffic Tool Works?

This tool will allow the user to create a set of APIs and applications in WSO2 API Manager according to a given scenario.  In addition to this, it will signup a group of users in WSO2 carbon and set them with subscriber privileges. These user details are randomly generated. The users should be distributed among applications according to a scenario and the traffic tool will generate access tokens for each user-application combination using the password grant type (see more).
 
 The traffic tool will continuously send traffic to the WSO2 API Manager throughout a user-specified time according to the given pattern.
 The attack tool will attack the WSO2 API Manager throughout a user-specified time. The attack tool is capable of simulating the following attack types.
 
 * **Denial of Service (DOS) attacks** - The DOS attacks aim to make the API resources unavailable for the intended users. As DOS attacks originate from a single source, the attack tool simulates DOS attacks by sending bursts of API requests to an API, using a single user who has subscribed to that particular API. Therefore, user IP, access token, and user cookie will stay the same for every request for a specific API.
 
 * **Distributed Denial of Service (DDOS) attacks** - The aim of the DDOS attacks is also to make the API resources unavailable for the intended users. The main difference between DOS and DDOS is that the DDOS attacks can originate from multiple sources, the attack tool simulates DDOS attacks by sending bursts of API requests to an API, using multiple users who have subscribed to that particular API. Therefore, user IP, access token, and user cookie will not be the same for every request for a particular API.
 
 * **Abnormal Token Usage Attacks** - Abnormal token usage can occur where the client's behavior deviates significantly from his usual behavior. For example, a user who rarely uses the cricket application mentioned in the example scenario suddenly starts to use the News and Cricket APIs in the cricket application heavily. The attack tool simulates this attack type by scaling up or down the request count in the usual traffic pattern for a particular period. (If the user sends only 5 GET requests per hour, the attack tool may send 50 requests in the attack duration)
 
 * **Extreme Delete Attacks** - In extreme delete attacks, an unusual number of DELETE requests are sent to an API resource. And also they usually occur without prior communication with the API (like without a GET request). The attack tool simulates extreme delete attacks by sending random requests to the DELETE endpoints of an API.
 
 * **Stolen Token Attacks** - In stolen token attacks, the attackers invoke APIs with access tokens that are stolen or hijacked. The attack tool simulates the stolen token attacks by sending API requests with valid access tokens but using IP addresses, user cookies, and user agents which are different from the normal invoke pattern.
