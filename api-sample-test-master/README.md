# API Sample Test

## Getting Started

This project requires a newer version of Node. Don't forget to install the NPM packages afterwards.

You should change the name of the `.env.example` file to `.env`.

Run `node app.js` to get things started. Hopefully the project should start without any errors.

## Explanations

The actual task will be explained separately.

This is a very simple project that pulls data from HubSpot's CRM API. It pulls and processes company and contact data from HubSpot but does not insert it into the database.

In HubSpot, contacts can be part of companies. HubSpot calls this relationship an association. That is, a contact has an association with a company. We make a separate call when processing contacts to fetch this association data.

The Domain model is a record signifying a HockeyStack customer. You shouldn't worry about the actual implementation of it. The only important property is the `hubspot`object in `integrations`. This is how we know which HubSpot instance to connect to.

The implementation of the server and the `server.js` is not important for this project.

Every data source in this project was created for test purposes. If any request takes more than 5 seconds to execute, there is something wrong with the implementation.

## Task

There is an engagement type called meeting in HubSpot. When a salesperson meets with a contact, they create a meeting through HubSpot for it.

Just like how we've done it for contacts and companies, write a method in the worker.js file that pulls and processes the meetings. Pull the meeting title and some timestamp properties from the meeting object. This worker will ideally run on a daily basis to get newly modified meetings.

You need to insert two actions as a result of this processing: Meeting Created (when that meeting record was created in HubSpot) and Meeting Updated (whenever it isn’t a Created action).

Store which contact attended the meeting along with the meeting properties. HubSpot normally doesn't give this data at the same time with the meetings so you should find another way to get it. At the end, you should have each meeting as a separate action along with the email of the contact which attended the meeting.

## Summary

My approach was based on integrating with an existing and running service. I intentionally avoided making changes to unrelated parts of the code to reduce the risk of introducing bugs. This decision was made to prioritize delivering the feature quickly and providing immediate business value. I've implemented the processMeetings method following the pattern of the other methods while incorporating a few targeted improvements.

## Debrief

### Code Quality and Readability

- The project would benefit from a rewrite in TypeScript to prevent type errors and improve maintainability. Additionally, mapping action types would provide clarity about the required data structure, helping new developers understand the system and reducing potential errors caused by "JavaScript guessing."

- In order to improve the code quality and readbility, The worker.js should be refactored focusing on separation of concerns. The file currently handles too many responsibilities, making it large, complex, and challenging to maintain. Refactoring the code into smaller, well-defined modules would be beneficial.

- Once we have a dedicated HubSpot Service, we could encapsulate crucial concerns such as security and rate limiting. This ensures that the rest of the code remains focused on business logic and does not need to handle API-level complexities.

- Improve the error handling by implementing a more structured error handling logic. Creating a centralized error managment module to help us easily identify where the error comes from and also give some context.

- The existing implementation of methods like processContacts and processCompanies retrieves the account object by searching the domain.integrations.hubspot.accounts array. This logic can be simplified by passing the account object directly as a parameter. While this change does not significantly impact performance, it removes unnecessary logic and makes the methods more testable.

### Project Architecture

- The three entities being processed (contacts, companies, and meetings) are currently independent. To optimize performance, we could split this job into multiple parallel workers, each processing one entity. This approach would significantly reduce processing time and distribute the load more efficiently.

- In a scenario where the system needs to scale to handle billions of inserts, I would recommend adopting message queues such as AWS SQS. This tool would decouple data ingestion from processing, allowing for distributed workloads and improving fault tolerance.

- In the context of processing large-scale data, I would advocate for a modular pipeline design. By breaking the processing into multiple steps (like a ETL). Each module could operate independently, allowing for better error isolation and easier upgrades.

- Monitoring tools could be integrated to track the health and performance of the worker. Metrics like API call success/failure rates, processing times, and queue sizes would provide valuable insights into system behavior and potential bottlenecks.

### Code Performance

- Implement a non linear pagination. Currently, all the methods uses offsetObject.after for linear pagination, which works well for smaller datasets. However, in scenarios involving large batches, a dynamic or parallel batch processing approach could significantly improve performance.

- In the current processMeetings method, meeting owners and contacts are fetched repeatedly. To address this, I implemented a simple in-memory cache mechanism to reuse already fetched entries, reducing redundant API calls and improving performance. For larger workloads or distributed environments, an external caching service like Redis should be considered to ensure reliability and scalability.

- Use a Promise.all call wrapping processContacts, processCompanies, processMeetings methods to reduce the execution time.
