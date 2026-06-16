After understanding the architecture of the application at the conceptual level and its services, i.e understanding exactly what we are building or what is being built, It is time for me to draw out the data flow diagram according to my architectural understanding and add trust boundaries, data flows and threat model the system. Here, I will draw out the elements, data flows, trust boundaries etc. The image below represents Level 0 data flow diagram of the overall system architecture representing all the elements, data flow and trust boundaries that make up the application.

Thinking about my data flow diagram more deeply, I realized that a threat actor may not necessarily interact with the system services directly via a web client and may in fact use a programmatic API client to interact with the services. If the micro services are exposed directly to the internet such that the web browser calls their endpoints directly with no API gateway sitting in between them, a threat actor can launch a proxy tool and intercept all the network traffic going through the web ingress and each micro service and organize them using a tool such as postman or a script, then use the tool to interact directly with the micro services with no line of defense between the tool and the micro services. This means that the web ingress client cannot act as a security boundary between the threat actor and the micro services. This will then open up a different attack surface than the one from the web ingress itself, therefore, this API client needs to be treated as an external service and be threat modeled appropriately. My diagram will then be updated to include the API Client in the untrusted zone as an external entity so we can threat model it appropriately. Of course, the API Client represents any API tool that can be used interact with the micro service endpoints in a programmatic way, such as a shell script, curl, custom bot, python script. So API Client there refers to a category of these. 

The diagram below represents the updated diagram.

![[updated-architecture.png]]

The architecture data flow diagram files can also be found in the diagrams folder in the globals folder.
## Modeling Threats Against The Data Flows, Elements and Trust Boundaries

Now that I have my architecture data flow diagram, I can now begin to identify threats in the system by running STRIDE against the data flows, elements and the trust boundaries **specifically analyzing what can go wrong in the architecture especially where trust changes across the trust boundaries or where data moves from an untrusted zone to a trusted zone** and what we can do about the threats to the system thereby essentially mitigating them. 

However, I want to specify some general table definition for the threat model which are the threat actors table that can have some interest to the system itself, the the data flows, assets table and element tables for reference sake etc. The tables act as a general reference point

The major goal of threat modeling the system is so we can identify what can go wrong at the systems level, what trust boundaries should exist, what our assets are that will be of great interest to a threat actor and what mitigations could prevent the threats from materializing against the system in the first place.

Because I use STRIDE against this initial architecture, subsequent Level 1 data flow diagrams that go deeper into each service and their trust boundaries will use OWASP API TOP 10 library of threats alongside STRIDE to make my threat identification for the analyzed service to be even more tactical.
### System Elements

Before I identify the key assets which are things of value in the system, first I want to flesh out the elements or components that make up the overall system itself.

The table below fleshes out all the system elements and their functionalities

| Element ID | Element Name             | Element Type    | Trust Zone               | Technology | Owner         | Description                                                                                          |
| ---------- | ------------------------ | --------------- | ------------------------ | ---------- | ------------- | ---------------------------------------------------------------------------------------------------- |
| **EL-01**  | Web Ingress Service      | Client          | External/Internet        | React      | Frontend Team | The main customer facing application. This is the main entry point for the users of the application. |
| **EL-02**  | API Client               | Client          | External/Internet        | Any        | Backend Team  | Bots, Programs can interact directly with the APIs if they are able to locate it.                    |
| **EL-03**  | Identity Service         | Process         | Internal (TB-01)         | Java       | Backend Team  | Authentication, User, Vehicle service responsible for handling user, auth and vehicle creation.      |
| **EL-04**  | Vehicle Workshop Service | Process         | Internal (TB-01)         | Python     | Backend Team  | Vehicle workshop service that handles mechanic, merchants, shop, orders and booking vehicle repairs  |
| **EL-05**  | Community Service        | Process         | Internal (TB-01)         | Go         | Backend Team  | Community service responsible for handling blog posts and comments                                   |
| **EL-06**  | Chatbot Service          | Process         | Internal (TB-01)         | Python     | Backend Team  | The chatbot service responsible for handling LLM service in the system                               |
| **EL-07**  | Gateway Service          | Process         | Internal (TB-01)         | Go         | Backend Team  | This service seems to look like an abandoned or experimental gateway service                         |
| **EL-08**  | Mailhog Service          | External System | Identity Service (TB-03) | Mailhog    | Mailhog       | The mailhog service responsible for handling OTP codes, auth codes from the identity service         |
| **EL-09**  | MongoDB                  | Data Store      | Internal (TB-02)         | MongoDB    | Ops Team      | The mongoDB database                                                                                 |
| **EL-10**  | PostgreSQL               | Data Store      | Internal (TB-02)         | PostgreSQL | Ops Team      | The postgresql database                                                                              |
| **EL-11**  | ChromaDB                 | Data Store      | Internal (TB-02)         | ChromaDB   | Ops Team      | The chromaDB database                                                                                |

### Trust Boundaries
Next, I want to analyze the trust boundaries that exists within the system...the place where data enters and exits and the trust zones where the data touches. This trust boundary explains where certain levels of trust exists and how the trust zones explains how the data is treated. The trust boundary determines where trust level changes.
### The System Assets
In order to model the threat actors who will have malicious interest in the system or maliciously influence the system to do things that it was not designed or intended to do, I first need to understand what assets exist on our system in the first place...what are those things of value that the threat actors would like to gain access to or steal from the system.

This section lists out the assets in the entire system in a table of system assets and grouped by each service. For each service, I want to analyze the assets that exists in them so that we can properly rank them, prioritize their value, risk and why a threat actor might want to lay their hands upon them.

To remind myself again of what the system does exactly or the value proposition of the system, first let's understand what we are dealing with again. *The system provides a platform for the servicing of vehicles/cars by selected choice of many mechanics and car parts accessories dealership and sales to users. Users come on to the platform upload their vehicles, look for a suitable mechanic and assign the repairs or servicing of their cars/vehicles to him/her. The users may also purchase car/vehicle parts and accessories from merchants who already exist on the platform and have been created by the admin of the platform. To make the platform engaging and not just strictly for vehicle servicing and car parts purchases, there is a mini blog platform where users can post blogs and comment on their blog posts and other user's blog posts too.* Basically, understanding this key functionalities will make it easier for me to identify the **Assets** that exist and subsequently the types of threat actors that would love to lay their hands on or steal the assets. This will help me in identifying the threats to the assets and the overall system.

In order to locate the assets in the overall system that threat actors would love to lay their hands on or see as value, I need to go deeper into the system specification itself which in this case is the documentation of the API contract that the code itself will eventually implement. My thinking is to extract what seem to be of value from the specification itself and layout the assets. I mean the overview doc, architecture diagram and docker compose file have already told me the key services and components in the overall system which are also the key services in the overall system,...However, I want to see the system specification contract itself which defines how the system is meant to be interacted with and what data it collects through which I can find the assets of value.

The specification for the system uses an Open API json documentation which I will load in a swagger editor online or via a vscode extension.

**The most important thing here is how the assets represent the key things that is valuable to the company, the thing of value that is of value to the users themselves and the thing of value that it is of value to the system or platform itself. Finding all these are very important as this is what threat actors or attackers would love to lay their hands on.**

Looking deeply at our system, the best place to identify and enumerate the assets will be in the design specification and we have this in the form of the Open API spec.

The table below summarizes the Assets that exists in the system.

#### Assets Table

| Asset ID  | Asset Name                  | Asset Type | Owner Service                | Description                                                                                        | Notes                                                                                                                                                                                                                       | Business Impact | C   | I   | A   |
| --------- | --------------------------- | ---------- | ---------------------------- | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | --- | --- | --- |
| **AS-01** | JWT Token                   | Secret     | Identity Service             | The JWT token implementation and handling. Secrets...etc.                                          | Implemented by the Identity service and issued to the user.                                                                                                                                                                 | Critical        | H   | H   | L   |
| **AS-02** | User PII                    | Data       | Identity & Community Service | User personal information like login email, phone number etc.                                      | The user's email address and phone number can be used as part of an authentication attack. Their profile details can contain personal information which can be of value to a threat actor.                                  | Medium          | H   | M   | H   |
| **AS-03** | Merchant & Mechanic Details | Data       | Vehicle Workshop Service     | The Vehicle workshop service dedicated to handling the merchants, mechanics, orders etc...         | The system's merchants, mechanics profile details and PII can be of value to a competitor therefore it is an asset.                                                                                                         | High            | H   | L   | L   |
| **AS-04** | Vehicle Details             | Data       | Vehicle Workshop Service     | Vehicle details belonging to a registered user.                                                    | The vehicle workshop details belonging to a user can contain critical vehicle details like the vin, pincode, owner details, vehicle location that can be of damage to the user if exposed.                                  | Low             | H   | L   | L   |
| **AS-05** | Coupon Implementation       | Data       | Community Service            | The coupon used when making the purchase of a vehicle accessory.                                   | If I am able to reuse a single coupon multiple times during a purchase, this could seriously impact the business revenue                                                                                                    | High            | H   | L   | L   |
| **AS-06** | Vehicle Accessories         | Data       | Vehicle Workshop Service     | The vehicle accessories available for purchase from a merchant                                     | If the system makes all the products available including the unavailable ones and doesn't protect the buying process, we can have scalping.                                                                                 | Medium          | M   | L   | H   |
| **AS-07** | Admin endpoints             | Secret     | Admin API endpoints          | The endpoints contain admin APIs used to manage merchants, mechanics and overall system operations | The system provides API services for consumption by its web application...However, If threat actors are able to find the admin endpoints which are not intended to be made public, then they have a critical attack surface | High            | H   | L   | L   |

### The Threat Actors 
Based on the assets found or discovered in the system, this section defines the type of threat actors that would want to lay their hands on the the assets...the threat actors that would be of interest in attacking our system...for example APTs won't necessarily have interest in our system and a competitor offering same service could be a threat...This table contains a list of threat actors that can attack, have interest in our system or maliciously influence our systems to do things we didn't intend the system to do.

In order to understand the type of threat actors that will have interest in our entire system, we need to know why they may need to have interest in our system and what they seek to achieve, gain or value to extract, this will be broken down better in the **Assets Section**. 

To remind me again, _we are a platform which provides vehicle servicing by mechanics and car parts accessories sales to users. We also allow users to blog and interact with each other. So what exactly will a threat actor or threat actors seek to gain or extract from our system. I will analyze that properly in the **Assets Section**_. I'd say that the type of threat actors we will have and model and will have interest in our system should be directly correlated and proportional to the ***Assets** our system seem to expose*.

Now that we have defined and enumerated our assets, i.e the things of value that our system, users and business actually value, we can begin enumerating the type of threat actors that would want to lay their hands on these assets. The types of attackers that would really care about the system's assets. This way, the threat actors will reveal themselves easily. 

The table below lists all the threat actors who will have interest in our assets, the table describes who they are, what their capabilities are and what they actually want.

| Actor ID  | Threat Actor      | Category           | Motivation            | Capability | Access Level                       | Target Assets                     |
| --------- | ----------------- | ------------------ | --------------------- | ---------- | ---------------------------------- | --------------------------------- |
| **TA-01** | External Attacker | Cybercriminal      | Financial Gain        | Medium     | Internet/Unauthenticated           | AS-01, AS-02, AS-04, AS-05, AS-07 |
| **TA-02** | Competitor        | Business Adversary | Competitive Advantage | Medium     | Internet/Unauthenticated, External | AS-02, AS-03, AS-07               |
| **TA-03** | Malicious Insider | Insider            | Fraud, Revenge        | High       | Privileged                         | AS-03, AS-04, AS-07, AS-05        |
| **TA-04** | Script Kiddie     | Opportunistic      | Curiosity, Reputation | Low        | Internet/Unauthenticated           | AS-02, AS-07                      |
### Data Flows
Now, we can analyze the data flows within the system. The data flows from a source element and from a trust zone to another trust boundary. The table below fleshes out the data flows that exists within the system, How data flows in, where it gets processed and how it is handled, rests/stored and how it exits...i.e its exit path.


