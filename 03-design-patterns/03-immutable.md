# Immutable Object Pattern
It is a creational pattern. It is used to create simple objects that can be shared across multiple classes, but for security reasons we don’t want their value to be modified.

Rules for implementing Immutable Object Pattern:
* Use a constructor to set all properties of the object.
* Mark all of the instance variables private and final.
* Don’t define any setter methods.
* Don’t allow referenced mutable objects to be modified or accessed directly.
* Prevent methods from being overridden.

Example:
```java
import java.util.*
public final class Animal {
   private final String species;
   private final int age;
   private final List favoriteFoods;
   public Animal(String species, int age, List favoriteFoods) {
      this.species = species;
      this.age = age;
      if(favoriteFoods == null) {
         throw new RuntimeException("favoriteFoods is required");
      }
      this.favoriteFoods = new ArrayList(favoriteFoods);
   }
   public String getSpecies() { return species; }
   public int getAge() { return age; }
   public int getFavoriteFoodsCount() { return favoriteFoods.size(); }
   public String getFavoriteFood(int index) { return favoriteFoods.get(index); }
}
```