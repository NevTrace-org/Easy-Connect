![Easy Connect logo](easyconnect-logo.jpg)

# Easy Connect - Blockchain for everyone

## 1. Project overview

Easy Connect is a solution that provides quick and easy access to any Qubic smart contract data, enabling applications and developers to leverage the full potential of this network into their business logic, without the need for them to implement any technical integration with the Qubic network.

In addition to providing quick integration for any application or service, Easy Connect provides a simple way for integrating Qubic into no-code platforms like Make or Zappier, allowing millions of users inmediate access to Qubic without requiring technical knowledge.

### 1.1 Example Use Case – Real-Time Alerts from QX Contract

Let’s say **Alice** wants to track market activity on the **Qubic decentralized exchange (QX contract)**. Specifically, she wants to know whenever someone places a bid to buy the token **MSVAULT**.

![ChatGPT Image 14 abr 2025, 12_55_38](https://github.com/user-attachments/assets/1967deac-0c51-4615-877f-29347ddaed69)

Using the Easy Connect web interface, Alice creates a new alert:

- **Contract**: QX  
- **Condition**: Bid for token  
- **Token**: MSVAULT  
- **Webhook URL**: *[her automation endpoint]*

Once she clicks **"Create Alert"**, Easy Connect configures the Qubic Integration Layer to monitor incoming transactions on the QX contract using the official **Qubic RPC interface**.

![ChatGPT Image 14 abr 2025, 12_59_15](https://github.com/user-attachments/assets/fe4aef95-4c48-4308-af6a-aa41a2acf0fa)

#### 1.2 Behind the scenes:

1. Easy Connect listens for transactions of type `AddToBidOrder`.
2. When a transaction matches the alert (e.g. a user places a bid to buy MSVAULT), Easy Connect decodes the binary payload received via RPC.
3. The decoded data includes key fields such as:

   | Field            | Value                            |
   |------------------|----------------------------------|
   | Procedure        | AddToBidOrder                    |
   | Token            | MSVAULT                          |
   | Price            | 150000001                        |
   | Quantity         | 1                                |
   | Issuer Address   | AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA |

4. If the token matches Alice’s condition (i.e., MSVAULT), Easy Connect sends this structured data to her defined webhook.
5. That webhook triggers an automation workflow via **Make**, where the data is stored in **Google Sheets**, sent to **Slack**, or integrated with any other service.

![ChatGPT Image 14 abr 2025, 13_03_07](https://github.com/user-attachments/assets/1daf8f1c-23ff-446a-83f5-94e307b212f5)

---

This MSVAULT bid tracking example will be referenced throughout the documentation to show how Easy Connect works in real scenarios.



## 2. Scope of the proposal

Our proposal for joining the Qubic incubation program is to create an initial version of Easy Connect that will monitor every information related to the main smart contracts already deployed into Qubic, allowing any application or service to subscribe and receive in real time every data generated on the network, via webhooks.

This first Easy Connect version will also offer a web-based configuration panel where users can select the smart contracts they want to monitor and the information they want to receive, as well as the endpoint to which our platform will send the data in real time.

The scope of the proposal includes the creation of a webhook-based Easy Connect integration example with Make, with the necessary documentation to allow any user, without technical knowledge, to integrate existing Qubic information into any application or process. Additionally, we will submit a request to Make to add Easy Connect to the more than 1,600 applications compatible with this leading no-code platform.


## 3. Functionality

### 3.1 Event abstraction from Qubic smart contracts
Easy Connect provides a real-time abstraction layer over the Qubic network, enabling applications and services to subscribe to smart contract activity without requiring low-level blockchain integration.

*For example, when a new `AddToBidOrder` is submitted to the QX contract (as in Alice’s case), Easy Connect handles the decoding and streamlines it into actionable data.*

![ChatGPT Image 14 abr 2025, 13_22_25](https://github.com/user-attachments/assets/7cf2538d-8d4e-40d5-bd5f-f6be7b60cb61)

### 3.2 Programmable alert system
Users can define customized conditions based on smart contract procedures, transaction parameters, sender/receiver addresses, and more. When these conditions are met, Easy Connect triggers alerts and initiates automated responses.

*In our example, Alice configures an alert for any bid on the token `MSVAULT`. When such a bid occurs, Easy Connect detects and processes it according to her defined rules.*

![ChatGPT Image 14 abr 2025, 13_29_40](https://github.com/user-attachments/assets/357188cf-08b6-4dba-87f4-ff532b052879)

### 3.3 Webhook-based data delivery
Decoded contract data is sent to user-defined endpoints via standard webhooks, allowing instant integration with automation platforms, third-party systems, or internal tools, without the need to maintain infrastructure.

*Alice uses a webhook URL linked to Make.com, which inserts the matching event directly into a Google Sheet. This integration requires no backend coding on her part.*

![ChatGPT Image 14 abr 2025, 13_25_09](https://github.com/user-attachments/assets/018a5415-1a9c-4775-999e-5df7aff19e9b)

### 3.4 Secure API access
Access to Easy Connect’s services is protected through an API key system, ensuring that only authorized users and systems can interact with the platform, and enabling usage monitoring per account.

*Alice’s webhook and alert configuration are both secured via her authenticated API token, ensuring that only her account can receive the alerts and make updates.*

![ChatGPT Image 14 abr 2025, 13_25_49](https://github.com/user-attachments/assets/65a7c503-72e6-44bb-90a3-a5d56ae47287)

### 3.5 Web-based management dashboard
A web interface allows users to manage their alerts, monitor activity, and review historical event data. Through this panel, users can activate or deactivate triggers, inspect past alerts, and configure webhook destinations.

*Alice creates and manages her alert for MSVAULT bids through this dashboard, where she can also review all recent matching events.*

![ChatGPT Image 14 abr 2025, 13_26_45](https://github.com/user-attachments/assets/9c9f4ee3-aafd-46fa-ad44-7abc2775bc74)

### 3.6 Authenticated user experience
Access to the management dashboard is gated by a simple, secure authentication system, ensuring that each user’s configuration and data remain isolated and protected within their account.

*Every action Alice takes—creating alerts, viewing history, editing destinations—is scoped to her secure session, isolating her data from others.*

![ChatGPT Image 14 abr 2025, 13_27_29](https://github.com/user-attachments/assets/9e05c866-f420-4360-8ab8-cb759f9fe264)

### 3.7 Roadmap

Future releases of Easy Connect plan to include additional features beyond the scope of the current proposal:
- Enable the execution of smart contract functions in Qubic through parameterization in the dashboard.
- Feature to read and decode transactions from any smart contract on the Qubic network.
- Develop and publish Easy Connect on other no-code platforms (e.g., Zappier).
- Integration with payment gateways (e.g., Stripe) to facilitate monetization.
- High availability architecture.

## 4. Tech stack

Easy Connect is architected as a cloud-native, event-driven platform optimized for scalability, maintainability, and real-time data processing. The solution leverages proven technologies to ensure high availability, secure access, and seamless integration with third-party systems.

![ChatGPT Image 14 abr 2025, 13_36_43](https://github.com/user-attachments/assets/bb0e6b0d-e9aa-4f9b-aa1f-ee0887b2d9b6)

### 4.1 Backend

The backend is developed in C# and deployed using AWS Lambda, following a serverless architecture to ensure elasticity and cost-efficiency. Event processing is decoupled using Amazon SQS, allowing for asynchronous and fault-tolerant handling of transaction streams, alert evaluation, and webhook dispatching.

### 4.2 Frontend

The user interface is built with Angular, providing a modern, responsive web application where users can authenticate, manage alerts, view their API key, and inspect historical events. The UI is designed to be intuitive for both developers and no-code users.

### 4.3 Database

A PostgreSQL instance serves as the main data store, managing user accounts, alert definitions, system events, and webhook delivery logs. It supports transactional integrity, complex filtering, and analytics over time-based blockchain activity.

### 4.4 Qubic integration layer

Implemented in C#, this component interfaces directly with the Qubic's RPC network to monitor activity on selected smart contracts and conditions. It decodes the current transaction's payloads into structured data models and publishes decoded events into Amazon SQS queues for further processing by downstream services.

### 4.5 Webhook and automation layer

Also developed in C#, the automation layer consumes events from SQS, evaluates alert conditions, and dispatches structured payloads to user-defined endpoints via HTTP webhooks. This makes it easy to integrate Qubic data with automation platforms like Make, Zapier, or enterprise systems.

## 5. Monetization

The strategy is based on launching with a freemium model to grow in users, who can upgrade to the Premium model to obtain additional functionalities, to generate income.

Easy Connect's business model will be based on two distinct customer segments:

### 5.1 Qubic ecosystem

The target audience is developers and users of the Qubic ecosystem who want to easily and quickly integrate real time data into their projects.

- **Free tier**: Monitoring of existing information in a single Qubic smart contract, with limited performance and basic support.
- **Premium tier**: Full access to the main Qubic smart contracts, parameterization through a user dashboard, receipt of real-time information and having premium support.

### 5.2 No-code platforms

Easy Connect Qubic modules for the main no-code platforms (Make, Zappier, etc.), offering two modalities:
- **Free tier**: Basic functionality, to use Easy Connect integration via webhook.
- **Premium tier**: Extended usage, advanced features, and premium support.  Available in future releases.

### 5.3 Early adoption incentives 

During the final phase of the project, we will launch campaigns aimed at attracting early users, incentivizing them through discounts and promotions, to scale faster after the launch and obtain promoters that will allow us to attract more users.
 
- **Discounts**: Reduced pricing for early users.  
- **Testimonial rewards**: Benefits for users sharing case studies.  
- **Roadmap influence**: Premium early users can vote on feature prioritization.


## 6. Marketing and communication strategy

### 6.1 Value proposition  

Position **Easy Connect as "the bridge between two worlds"**, the intricate blockchain ecosystem and the user-friendly automation tools businesses already rely on.

Core messaging will emphasize:  
- **Democratization**: "Qubic for everyone, no coding required."  
- **Simplicity**: "Transform weeks of development into minutes of configuration."  
- **Trust**: "Deliver trustworthy Qubic data to business processes." 

### 6.2 Audience targeting

We will focus on three different buyer persona:
- **Qubic ecosystem**: Existing applications and services who want to obtain data in real time for monitoring, reporting, user support...
- **No-code/low-code developers**: Current users of Make/Zapier looking to expand their automation capabilities including Qubic data.  
- **Web3 startups and blockchain firms**: Requiring seamless integration with Qubic network.  

### 6.3 Social media strategy  

Since we won second prize at [MAD Hack 2025](https://qubic.org/blog-detail/qubic-vottun-2025-madrid-hackathon-recap) we have started a social media positioning strategy, which will be increased during the development of the project to achieve community engagement and capture potential leads, as well as Easy Connect promoters.

We will focus on X and Farcaster, where the communities interested in Blockchain and Web3 are currently located, to capture potential leads and generate engagement.

In parallel, we will use LinkedIn to reach companies and professionals interested in automation and the Blockchain applications they can integrate into their businesses.

We will continue to actively participate in the Qubic ecosystem (in English and Spanish) on both Telegram and Discord, to achieve community engagement and capture potential leads, as well as promoters.

If we have the budget available, we will define additional actions to promote Easy Connect.

### 6.4 Implementation timeline 

The marketing and communications strategy will be developed in parallel with the execution of the Easy Connect project plan and will continue beyond the scope indicated in this proposal. 


## 7. Project plan and deliverables

Project planning is structured into the following phases, each of which specifies the activities and deliverables to be generated, start date, duration and dedication in hours per profile:

**Phase 0 - Project proposal:** 
- Start date (MAD HACK 2025): March 24 2025
- Committee aproval date (T1): April 16 2025 (TBC)

**Phase 1 - Technical design:** 
- Duration: 5 weeks + 1 week (Qubic QA)
- Scope: 
  - Architecture definition.
  - Functional definition.
  - Design definition (Figma wireframes).
  - Marketing and communication plan.
- Dedication:
  - Development: 75 hours
  - UX/UI: 25 hours
  - Project management / Business development / Marketing and communication: 37,5 hours

**Phase 2 - Testnet integration:** 
- Duration: 6 weeks + 1 week (Qubic QA)
- Scope:
  - Qubic integration with one smart contract (test environment).
  - Webhook integration beta.
  - Initial integration tests.
  - Landing page beta.
  - Management dashboard beta.
  - Community building.
- Dedication:
  - Development: 120 hours
  - UX/UI: 45 hours
  - Project management / Business development / Marketing and communication: 30 hours

**Phase 3 - Mainnet integration:**
- Duration: 5 weeks + 1 week (Qubic QA)
- Scope:
  - Qubic integration with most relevant mainnet smart contracts.
  - Webhook integration v1.0.
  - Landing page v1.0.
  - Management dashboard v1.0.
  - Integration tests.
  - Initial marketing campaigns (soft launch).
- Dedication:
  - Development: 100 hours
  - UX/UI: 25 hours
  - Project management / Marketing and communication: 37,5 hours

**Phase 4 - Make integration (webhook):**
- Duration: 4 weeks + 1 week (Qubic QA)
- Scope:
  - Qubic Easy Connect example for Make.
  - Submission of custom module to Make (*)
  - Usability test.
  - Technical documentation.
  - User documentation.
  - Easy Connect roadmap.
- Dedication:
  - Development: 80 hours
  - UX/UI: 30 hours
  - Project management / Marketing and communication: 20 hours

This planning and scope is subject to the technical availability of the mainnet and testnet networks, technical documentation, and development tools provided by Qubic, as well as the support required for the development of the various components of the solution. 

(*) Submission to Make does not imply acceptation.

## 8. Budget and payment plan

### 8.1 Budget breakdown

The budget required for the development of the Easy Connect solution, according to the described scope, amounts to 25,000 EUR, broken down into the following items:
- **Development:** 15,000 EUR
- **UX/UI Design:** 5,000 EUR
- **Project management:** 2,500 EUR
- **Product and Business development:** 1,500 EUR
- **Marketing and Communications:** 1,000 EUR

### 8.2 Payment plan

Payments will be made according to the following schedule, based on the achievement of the specified milestones:
- **Milestone 0 - Proposal aproval:** 
  - Date: T1
  - Amount: 20% budget (5,000 EUR)
- **Milestone 1 - Technical design:**
  - Date: T1 + 6 weeks
  - Amount: 25% budget (6,250 EUR)
- **Milestone 2 - Testnet integration:**
  - Date: T1 + 13 weeks
  - Amount: 25% budget (6,250 EUR)
- **Milestone 3 - Mainnet integration:** 
  - Date: T1 + 19 weeks
  - Amount: 20% budget (5,000 EUR)
- **Milestone 4 - Make integration:**
  - Date: T1 + 24 weeks
  - Amount: 10% budget (2,500 EUR)

### 8.3 Disclaimer: Operational costs beyond project scope

Upon completion of the project outlined in this proposal, the operation of the Easy Connect solution will incur additional costs not covered in the proposed budget, including (but not limited to):
- Hosting
- Software and licenses
- Operation and maintenance
- Customer support
- Marketing
- Community management

We confirm our willingness to maintain and operate the Easy Connect solution beyond the completion of the project funded by the incubation program.

However, additional funding will be required if the revenue from Easy Connect's commercialization does not cover these costs, either through Qubic's grant programs or other financing alternatives.


## 9. Potential new project: C# SDK

As part of the Easy Connect project, we plan to create a basic C# SDK for Qubic’s RPC. This will serve as a foundational component of our Qubic integration layer and could later be expanded into a comprehensive RPC C# SDK.

After completing the Easy Connect project, we aim to submit a grant proposal to develop a fully functional open-source SDK. This initiative will support developers in integrating with Qubic using a broader range of programming languages.


## 10. Profile and experience

### 10.1 Team members

- **Jorge Ordovás (Product and Business Development / Marketing and Communications):**
  - Information Technology professional with 25 years of experience in the development of products and services in many different sectors (telecommunications, payments, security, eHealth, energy, cloud, blockchain, web3).
  - Working in Blockchain consulting and development of projects based on blockchain technologies since 2015 when he cofounded [Nevtrace](https://nevtrace.com), the first Blockchain lab in Spain.
  - Senior manager working in Blockchain and Web3 product and business development since 2018 at [Telefonica](https://metaverso.telefonica.com/en/welcome-to-metaverse).
  - **LinkedIn (+10,000 followers):** [Jorge Ordovas](https://www.linkedin.com/in/jorgeordovas/)
  - **X (+6,000 followers):** [@joobid](https://x.com/joobid) and [@nevtrace](https://x.com/nevtrace)
  - **Farcaster (+3,000 followers):** [@joobid.eth](https://warpcast.com/joobid.eth) and [@nevtrace](https://warpcast.com/nevtrace)

- **Rafael Montaño (UX/UI and Product Design)**
  - Co-founder of [Loiband](https://www.loiband.com/home-en), technology consultancy and UX Product Specialist, with experience in UX/UI and product design. He has led the creation of more than 10 scalable digital products.
  - His approach combines user research, prototyping, and design systems in apps, Web2, and Web3, helping startups and companies launch innovative and optimized solutions for the digital market.
  - **LinkedIn:** [Rafael Montaño](https://www.linkedin.com/in/rafael-monta%C3%B1o-marroquin/)
 
- **Jesús Lanzarote (Full Stack Development)**
  - Co-founder and CTO of Loiband. He has been programming for over 25 years and has worked in the startup world for 15 years, where he has participated in all types of projects across multiple areas (eHealth, services, insurance, legal, etc.), leading teams and directly participating in design and development.
  - His current focus is on developing AI-based and Web3 projects.
  - **LinkedIn:** [Jesús Lanzarote](https://www.linkedin.com/in/jesus-lanzarote/)

### 10.2 External collaborators

- **Pablo F. Burgueño (Advisor)**
  - Cofounder of NevTrace and a lawyer specializing in cybersecurity, privacy, and LegalTech law.
  - He currently works as a lawyer and professor at ESIC, and he is a member of INMERSIVA XR, DENAE, and ASEDA, as well as the data protection and cryptoasset spaces of the European Cybercrime Centre at Europol.
  - **LinkedIn (+13,000 followers):** [Pablo F. Burgueño](https://www.linkedin.com/in/pablofb/)
  - **X (+8,000 followers):** [@pablofb](https://x.com/pablofb)
  - **Farcaster (+2,000 followers):** [@pablofb](https://warpcast.com/pablofb)

Moreover, with more than a decade of experience in the startup and corporate ecosystem, we have a broad network of contacts, freelance professionals specialized in blockchain technology, frontend, backend, infrastructure, and more. We will involve them in the project as external collaborators if additional resources are needed to meet the defined milestones.
  
### 10.3 Public projects developed by team members

* [Foodcoin](https://www.foodcoin.global)
* [Referity](https://referity.es)
* [Residelia](https://residelia.com)
* [Mscope](https://www.mscope.tech)
* [Heris](https://heris.loiband.com)
* [Normafin](https://normafin.com)
* [TU Wallet](https://www.telefonica.com/en/transformation-handbooks/innovation-handbook/tu-wallet-a-multi-crypto-digital-wallet/)
* [TrustOS](https://www.eleconomista.es/especial-tecnologia-startups/noticias/10261692/12/19/Telefonica-lanza-TrustOS-para-facilitar-operaciones-blockchain.html)


## 11. Conclusion

At NevTrace we've been working for a decade to **become the link between the real world and the on-chain ecosystem**.

Our mission is to help companies that want to take advantage of the opportunities offered by blockchain **and on-chain projects that need to go beyond early adopters**.

The Easy Connect proposal we're presenting **aligns perfectly with our vision and addresses a real need in the Qubic ecosystem** to drive its growth both within the current ecosystem and beyond, toward the early majority.

**We're confident that the proposal will be interesting for Qubic**, and that we can develop an initial commercial version of Easy Connect thanks to the Qubic Incubation Program.

Finally, **we confirm our commitment to maintaining and operating the Easy Connect solution after the completion of the project funded by the incubation program**. However, additional funding will be required if revenues from its commercialization do not cover the costs, either through Qubic's grant programs or other financing alternatives.
