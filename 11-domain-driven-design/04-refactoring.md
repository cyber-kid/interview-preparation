# Refactoring

* [POLICY](#policy)
* [SPECIFICATION](#specification)
* [SIDE-EFFECT-FREE FUNCTIONS](#side-effect-free-functions)
* [CLOSURE OF OPERATIONS](#closure-of-operations)

#### POLICY
Here are some warning signs that a constraint is distorting the design of its host object:
1. Evaluating a constraint requires data that does not otherwise fit the object’s definition.
2. Related rules appear in multiple objects, forcing duplication or inheritance between objects that are not otherwise a family.
3. A lot of design and requirements conversation revolves around the constraints, but in the implementation, they are hidden away in procedural code.

When the constraints are obscuring the object’s basic responsibility, or when the constraint is prominent in the domain yet not prominent in the model, you can factor it out into an explicit object or even model it as a set of objects and relationships.

#### SPECIFICATION
A SPECIFICATION states a constraint on the state of another object, which may or may not be present. It has multiple uses, but the most basic concept of it is that a SPECIFICATION can test any object to see if it satisfies the specified criteria.

Create explicit predicate-like VALUE OBJECTS for specialized purposes. A SPECIFICATION is a predicate that determines if an object does or does not satisfy some criteria.

Much of the value of SPECIFICATION is that it unifies application functionality that may seem quite different. We need to specify the state of an object for three reasons:
* validation of an object to see if it fulfills some need or is ready for some purpose;
* selection of an object from a collection;
* specifying the creation of a new object to fit some need;

#### SIDE-EFFECT-FREE FUNCTIONS
Place as much of the logic of the program as possible into functions, operations that return results with no observable side effects. Strictly segregate commands (resulting inmodifications to observable state) into very simple operations that do not return domain information. Further control side effects by moving complex logic into VALUE OBJECTS with conceptual definitions fitting the responsibility.

#### CLOSURE OF OPERATIONS
Where it fits, define an operation whose return type is the same as the type of its argument(s). If the implementer’s has state that is used in the computation, then the implementer is effectively an argument of the operation, so the argument(s) and return value should be of the same type as the implementer. Such an operation is closed under the set of instances of that type. A closed operation provides a high-level interface without introducing any dependency on other concepts.
