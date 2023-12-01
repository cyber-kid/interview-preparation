# Saga distributed transactions pattern

The Saga design pattern is a way to manage data consistency across microservices in distributed transaction scenarios. A saga is a sequence of transactions that updates each service and publishes a message or event to trigger the next transaction step. If a step fails, the saga executes compensating transactions that counteract the preceding transactions.

The Saga pattern provides transaction management using a sequence of local transactions. A local transaction is the atomic work effort performed by a saga participant. Each local transaction updates the database and publishes a message or event to trigger the next local transaction in the saga. If a local transaction fails, the saga executes a series of compensating transactions that undo the changes that were made by the preceding local transactions.

In Saga patterns:
* **Compensable transactions** are transactions that can potentially be reversed by processing another transaction with the opposite effect.
* A **pivot transaction** is the go/no-go point in a saga. If the pivot transaction commits, the saga runs until completion. A pivot transaction can be a transaction that is neither compensable nor retryable, or it can be the last compensable transaction or the first retryable transaction in the saga.
* **Retryable transactions** are transactions that follow the pivot transaction and are guaranteed to succeed.

In Sagas, every step in the workflow executes its portion of the work, registers a callback to a **compensating transaction** in a message called a **routing slip** and passes the updated message down the activity chain. If any step downstream fails, that step looks at the routing slip and invokes the most recent step’s compensating transaction, passing back the routing slip. The previous step does the same thing, calling its predecessor compensating transaction and so on until all already executed transactions are compensated.

There are two common saga implementation approaches, choreography and orchestration. Each approach has its own set of challenges and technologies to coordinate the workflow.

### Choreography
Choreography is a way to coordinate sagas where participants exchange events without a centralized point of control. With choreography, each local transaction publishes domain events that trigger local transactions in other services.

Benefits
* Good for simple workflows that require few participants and don't need a coordination logic.
* Doesn't require additional service implementation and maintenance.
* Doesn't introduce a single point of failure, since the responsibilities are distributed across the saga participants.

Drawbacks
* Workflow can become confusing when adding new steps, as it's difficult to track which saga participants listen to which commands.
* There's a risk of cyclic dependency between saga participants because they have to consume each other's commands.
* Integration testing is difficult because all services must be running to simulate a transaction.

### Orchestration
Orchestration is a way to coordinate sagas where a centralized controller tells the saga participants what local transactions to execute. The saga orchestrator handles all the transactions and tells the participants which operation to perform based on events. The orchestrator executes saga requests, stores and interprets the states of each task, and handles failure recovery with compensating transactions.

Benefits
* Good for complex workflows involving many participants or new participants added over time.
* Suitable when there is control over every participant in the process, and control over the flow of activities.
* Doesn't introduce cyclical dependencies, because the orchestrator unilaterally depends on the saga participants.
* Saga participants don't need to know about commands for other participants. Clear separation of concerns simplifies business logic.

Drawbacks
* Additional design complexity requires an implementation of a coordination logic.
* There's an additional point of failure, because the orchestrator manages the complete workflow.

Link: https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga