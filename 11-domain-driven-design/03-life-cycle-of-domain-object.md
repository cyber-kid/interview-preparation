# The Lifecycle of a Domain Object
* [AGGREGATES](#aggregates)
* [FACTORIES](#factories)
* [REPOSITORIES](#repositories)

The problems fall into two categories:
* Maintaining integrity throughout the lifecycle;
* Preventing the model from getting swamped by the complexity of managing the lifecycle;

Three patterns will address these issues. First, AGGREGATES tighten up the model itself by defining clear ownership and boundaries, avoiding a chaotic tangled web of objects. This is crucial to maintaining integrity in all phases of the lifecycle. Then, we focus on the beginning of the lifecycle, using FACTORIES to create and reconstitute complex objects and AGGREGATES, keeping their internal structure encapsulated. Finally, REPOSITORIES address the middle and end of the lifecycle, providing the means of finding and retrieving persistent objects while encapsulating the immense infrastructure involved.

#### AGGREGATES
It is difficult to guarantee the consistency of changes to objects in a model with complex associations. Invariants need to be maintained that apply to closely related groups of objects, not just discrete objects. Yet cautious locking schemes cause multiple users to interfere pointlessly with each other and make a system unusable.

An AGGREGATE is a cluster of associated objects that we treat as a unit for the purpose of data changes. Each AGGREGATE has a root and a boundary. The boundary defines what is inside the AGGREGATE. The root is a single specific ENTITY contained in the AGGREGATE. The root is the only member of the AGGREGATE that outside objects are allowed to hold references to, although objects within the boundary may hold references to each other. ENTITIES other than the root have local identity, but it only needs to be unique within the aggregate, since no outside object can ever see it out of the context of the root ENTITY.

A model of a car might be used in software for an auto repair shop. The car is an ENTITY with global identity – we want to distinguish that car from all other cars in the world, even very similar ones. We can use the Vehicle Identification Number for this, a unique identifier assigned to each new car.We might want to track the rotation history of the tires through the four wheel positions. We might want to know mileage and tread wear of each tire. To know which tire is which, the tires must be identified ENTITIES also. But it is very unlikely that we care about the identity of those tires outside of the context of that particular car. If we replace the tires and send the old ones to a recycling plant, either our software will no longer track them at all, or they will become anonymous members of a heap of tires. No one will care about their rotation histories. More to the point, even while they are attached to the car, no one will try to query the system to find a particular tire and then see which car it is on. They will query the database to find a car and then ask it for a transient reference to the tires. Therefore, the car is the root ENTITY of the AGGREGATE whose boundary encloses the tires also. On the other hand, engine blocks have serial numbers engraved on them and are sometimes tracked independently of the car. In some applications, the engine might be the root of its own AGGREGATE.

Invariants, consistency rules that must be maintained whenever data changes, will involve relationships between members of the AGGREGATE.

Now, to translate that conceptual AGGREGATE into the implementation, we need a set of rules to apply to all transactions:
* The root ENTITY has global identity, and is ultimately responsible for checking invariants.
* Root ENTITIES have global identity. ENTITIES inside the boundary have local identity, unique only within the AGGREGATE.
* Nothing outside the AGGREGATE boundary can hold a reference to anything inside, except to the root ENTITY. The root ENTITY can hand references to the internal ENTITIES to other objects, but those objects can only use them transiently, and may not hold onto the reference. The root may hand a copy of a value to another object, and it doesn’t matter what happens to it, since it’s just a value and no longer will have any association with the AGGREGATE.
* As a corollary to the above rule, only AGGREGATE roots can be obtained directly with database queries. All other objects must be found by traversal of associations.
* Objects within the AGGREGATE can hold references to other AGGREGATE roots.
* A delete operation must remove everything within the AGGREGATE boundary at once. (With garbage collection, this is easy. Since there are no outside references to anything but the root, delete the root and everything else will be collected.)
* When a change to any object within the AGGREGATE boundary is committed, all invariants of the whole AGGREGATE must be satisfied.

#### FACTORIES
Creation of an object can be a major operation in itself, but complex assembly operations do not fit the responsibility of the created objects. Combining these responsibilities can produce ungainly designs that are hard to understand. Shifting the responsibility of directing construction to the client muddies the design of the client, breaches encapsulation of the assembled object or AGGREGATE and overly couples the client to the implementation of the created object.

Just as the interface of an object should encapsulate its implementation, allowing a client to use its behavior without knowing how it works, a FACTORY encapsulates the knowledge needed to create a complex object or AGGREGATE. It provides an interface that reflects the goals of the client and an abstract view of the created object.

Shift the responsibility for creating instances of complex objects and AGGREGATES to a separate object, which may itself have no responsibility in the domain model, but is still part of the domain design. Provide an interface that encapsulates all complex assembly and that does not require the client to reference the concrete classes of the objects being instantiated. Create entire AGGREGATES as a piece, enforcing their invariants.

The two basic requirements for any good FACTORY are:
* Each creation method is atomic and enforces all invariants of the created object or AGGREGATE. It should only be able to produce an object in a consistent state. For an ENTITY this means the creation of the entire AGGREGATE, with all invariants satisfied, but probably with optional elements still to be added. For an immutable VALUE this means all attributes are initialized to their correct final state. If the interface makes it possible to request an object that can’t be created correctly, then an exception should be raised or some other mechanism should be invoked that will ensure that no improper return value is possible.
* The FACTORY should be abstracted to the type desired, rather than the concrete class(es) involved.

#### REPOSITORIES
A REPOSITORY represents all objects of a certain type as a conceptual set (usually simulated). It acts like a collection, except with more elaborate querying capability. Objects of the appropriate type are added and removed, and the machinery behind the REPOSITORY inserts them or deletes them from the database. This definition gathers a cohesive set of responsibilities for providing access to the roots of AGGREGATES from early-lifecycle through the end. Clients request objects from the REPOSITORY using query methods that select objects based on criteria specified by the client, typically specifying the value of certain attributes. The REPOSITORY retrieves the requested object, encapsulating the machinery of database queries and metadata mapping. Repositories can implement a variety of queries that select objects based on whatever criteria the client requires. They can also return summary information, such as a count of how many instances meet some criteria. They can even return summary calculations, such as the total across all matching objects of some numerical attribute.
