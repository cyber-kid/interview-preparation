# Builder Pattern
It is a creational pattern in which parameters are passed to a builder object by setter methods, often through method chaining, and an object is generated with a final build call.

Rules for implementing Builder Pattern:
* It is often used with immutable objects, since immutable objects do not have setter methods and must be created with all of their parameters set, although it can be used with mutable objects as well. But itself it is mutable.
* All of the setters return the instance of the builder object this.
* The target object is created with a build method usually named build(). This method passes all variables that were set by the builder to the underlying object`s constructor. The build() method can set default values for variables that were not provided by user.

Example:
```java
import java.util.*;
public class AnimalBuilder {
   private String species;
   private int age;
   private List favoriteFoods;

   public AnimalBuilder setAge(int age) { this.age = age; return this; }
   public AnimalBuilder setSpecies(String species) { this.species = species; return this; }
   public AnimalBuilder setFavoriteFoods(List favoriteFoods) { this.favoriteFoods = favoriteFoods; return this; }
  
   public Animal build() {
      return new Animal(species,age,favoriteFoods);
   }
}
```
In practice, a builder class is often packaged alongside its target class, either as a static inner class within the target class or within the same Java package.