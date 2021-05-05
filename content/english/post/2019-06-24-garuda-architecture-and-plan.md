---
title: Garuda – Architecture and Plan
author: Ravikanth C
type: epic
date: 2019-06-24T04:01:55+00:00
url: /2019/06/24/garuda-architecture-and-plan/
post_views_count:
  - 3303
views:
  - 3389
categories:
  - DevOps
  - Pester
  - Garuda
tags:
  - DevOps
  - Pester

---
In the first part of this [series](/categories/garuda), I mentioned the reasoning behind starting development of a new framework for operations validation. Towards the end, I introduced Garuda &#8212; a distributed and flexible operations validation framework. There are certain principles that drove the design of this framework &#8212; Distributed, Flexible, and Secure.

In this part of the series, you will see the architecture proposal and the plan I have to implement these features. 

### Architecture

To support the principles described, the framework needs a few moving parts. These moving parts provide the flexibility needed and gives you a choice of tooling.

{{< figure src="/images/garuda1.png" >}} {{< load-photoswipe >}}

At a high-level, there are five components in this framework. 

### Test Library

The test library is just a collection of parameterized Pester test scripts. The parameterization helps in reuse. As a part of the Garuda core, you can group these tests and then use the publish engine to push the tests to remote targets. The tags within the tests are used in the test execution process to control what tests need to be executed within the test group published to the remote target. 

### Garuda Core

This is the main module which sticks the remaining pieces together. This is what you can use to manage the test library. This core module will provide the ability to parse the test library and generate the parameter information for each test script which is eventually used in generating a parameter manifest or configuration data. One of the requirements for this framework is to enable grouping of tests. The Garuda core gives you the ability to generate the test groups based on what is there in the library. You can then generate the necessary parameter manifest (configuration data) template for the test group that you want to publish to remote targets. Once you have the configuration data prepared for the test group, you can publish the tests to the remote targets.

### Publish Engine

The publish engine is responsible for several things. 

This module generates the necessary configuration or deploy script that does the following:

  1. Install necessary dependent modules (for test scripts from a local repository or PowerShell Gallery)
  2. Copy test scripts from the selected test group to the remote target
  3. Copy the configuration data (sans the sensitive data) to the remote target 
  4. As needed, store credentials to a credential vault on the remote target
  5. Copy the Chakra engine to the remote targets
  6. if selected, create the JEA endpoints for operators to retrieve test results from the remote targets
  7. If selected, create scheduled tasks on the remote target for reoccurring test script execution

Once the configuration or the deploy script is ready, the publish engine can enact it directly on the remote targets or just return the script for you to enact it yourself.

The publish engine is extensible and by default will support PowerShell DSC and [PSDeploy](https://github.com/RamblingCookieMonster/PSDeploy) for publishing tests to the remote targets. Eventually, I hope the community will write providers for other configuration management platforms / tools. There will be abstractions within the engine to add these providers in a transparent manner.

The publish engine helps in securing the test execution by storing sensitive configuration data in a vault. It also implements the scheduled tasks as either SYSTEM account or a specified user. The JEA endpoints configured on the remote targets help us securely retrieve the test results with least privileges needed.

You can publish multiple tests groups to the same remote target. This helps implement the flexibility that IT teams need in the operations space for a given infrastructure or application workload. There can be multiple JEA endpoints one for each team publishing the test groups.

### Chakra

The Chakra is what helps execute the tests in test group(s) on the remote targets. It has the test parameter awareness and can use the published configuration data and the sensitive data stored in the vault for unattended execution of tests. Chakra is also responsible for result consolidation. It can be configured to retain results for X number of days. All the test results for each group get stored as JSON files and are always timestamped. The scheduled tasks created on the remote targets invoke Chakra at the specified intervals. Chakra also contains the cmdlets that are configured for access within the JEA endpoints. Using these endpoints, test operators can retrieve the test results from a central system from all the remote targets.

### Report Engine

The report engine is final piece of this framework that enables retrieving the results from the remote targets and transforming those results into something meaningful for the IT managers. By default, there will be providers for reports based on [PSHTML](https://github.com/Stephanevg/PSHTML), [ImportExcel](https://github.com/dfinke/ImportExcel), and [UniversalDashboard](https://github.com/ironmansoftware/universal-dashboard ). The report engine provides that abstractions for community to add more reporting options.

### The Plan

The initial release of the framework or what I demonstrated at the PowerShell Conference Europe was just a proof of concept. I am planning on breaking down the framework into different core components I mentioned above. The GitHub repository for the framework will have issues created for each of these components and I will start implementing the basics. The 1.0 release of this framework will have support for every detail mentioned above and will be completely useable and functional for production use.

### What about the naming?

The names _Garuda_ and _Chakra_ are from the Hindu mythology and their meaning is connected to the concepts I am proposing for this framework. Garuda is the [bird from Hindu mythology](https://en.wikipedia.org/wiki/Garuda ). It has a mix of human and bird features. It is deemed powerful and is the vehicle of the Hindu god Vishnu. It can travel anywhere and is considered the king of birds. The [Chakra](https://en.wikipedia.org/wiki/Sudarshana_Chakra) the weapon that lord Vishnu carries. It is used to eliminate evil. This is also known as the wheel of time. Garuda is the vehicle that transports lord Vishnu and his Chakra to places where there is evil.

The Garuda Core combined with the Publish engine can take your operational validation tests to your remote targets. Chakra is the way to perform operations validation to ensure that your infrastructure is always healthy and functional. 

In the next article in this series, you will see the framework in action.