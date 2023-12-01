# CQRS

CQRS stands for Command and Query Responsibility Segregation, a pattern that separates read and update operations for a data store. Implementing CQRS in your application can maximize its performance, scalability, and security. The flexibility created by migrating to CQRS allows a system to better evolve over time and prevents update commands from causing merge conflicts at the domain level.

CQRS separates reads and writes into different models, using commands to update data, and queries to read data.
* Commands should be task-based, rather than data centric. ("Book hotel room", not "set ReservationStatus to Reserved").
* Commands may be placed on a queue for asynchronous processing, rather than being processed synchronously.
* Queries never modify the database. A query returns a DTO that does not encapsulate any domain knowledge.


Having separate query and update models simplifies the design and implementation. However, one disadvantage is that CQRS code can't automatically be generated from a database schema using scaffolding mechanisms such as O/RM tools (However, you will be able to build your customization on top of the generated code).

For greater isolation, you can physically separate the read data from the write data. In that case, the read database can use its own data schema that is optimized for queries. For example, it can store a materialized view of the data, in order to avoid complex joins or complex O/RM mappings. It might even use a different type of data store. For example, the write database might be relational, while the read database is a document database.

Benefits of CQRS include:
* **Independent scaling** CQRS allows the read and write workloads to scale independently, and may result in fewer lock contentions.
* **Optimized data schemas** The read side can use a schema that is optimized for queries, while the write side uses a schema that is optimized for updates.
* **Security** It's easier to ensure that only the right domain entities are performing writes on the data.
* **Separation of concerns** Segregating the read and write sides can result in models that are more maintainable and flexible. Most of the complex business logic goes into the write model. The read model can be relatively simple.
* **Simpler queries** By storing a materialized view in the read database, the application can avoid complex joins when querying.

Link: https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs