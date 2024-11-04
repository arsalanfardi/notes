Object creation order:
- Static initializers are at class load time
- Super class is always completed before subclass 
- Member variables with initialization before constructor

Tip:
- Don't call a method that can be overwritten from a constructor
	- An object can be in an incomplete state when it's called, leading to NPEs
# Item 1: Consider static factory methods instead of constructors
- **Static factory method**: A static method that returns an instance of the class
	- Not to be confused with Factory design pattern
	- Below is an example from the *boxed primitive* class for boolean (Boolean)
```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

Java does this under the hood when converting primitives to box primitives (**autoboxing**).

```Java
public class Autobox {
	Integer foo() {
		return 7; // Integer.valueOf() is called
	}
}
```
### Advantages
1. **They have names**
- Constructor overloading is confusing for a user since each constructor has the same name
- Static factory methods do not have this naming restriction, and therefore can have more descriptive names

2. **They are not required to create a new object with each invocation**
- See for example `Boolean.valueOf` above
- Can use cached instances to avoid creating unnecessary duplicate objects
- Classes can be **instance-controlled**, meaning they have control over what instances exist at any time
	- Can guarantee:
		- Singletons
		- Noninstantiability 
		- No two equal instance exist, e.g. `a.equals(b)` only if `a == b` (enums provide this guarantee)

3. **They can return an object of any subtype of their return type**
- Added flexibility in choosing returned object
- Enables hiding implementation details (can return objects without making classes public)
- Think Collections framework:
	- `java.util.Collections` is a noninstantiable companion class, however these are no longer required after Java 8 since interfaces can contain static methods (e.g. `List.of(), Set.of()`)
```java
// Both of these methods hide the underlying implementation and only return a List Type
List<String> list = Collections.unmodifiableList(new ArrayList<>());
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
```

4. **The class of returned object can vary from call-to-call as a function of the input parameters**
- For example. `EnumSet` class only has static factories, and depending on the size of the underlying enum type it returns a different subclass instance (but this is invisible to clients)
	- <= 64 - `RegularEnumSet` (backed by a single long)
	- > 64 - `JumboEnumSet` (backed by a long array)

5. **The class of the returned object does not need to exist when the class containing the method is written**
- The basis of **service provider frameworks** like JDBC
	- These frameworks are a system where providers implement a service and the system makes the implementations available to the to clients
	- Enables a client to use different implementations of a service at **runtime** without needing to know specifics in advance
- Java provides the `java.util.ServiceLoader` framework since Java 6

### Limitations
1. **Only using static factory methods (without constructors) means that a class cannot be subclassed (although, this isn't necessarily a bad thing)**
- For example, none of the classes from `Collections` (`List`, `Set` can be subclassed)

2. **They can be harder to find in API documentation**
- Recommend using common names:
	- `from` - Type conversion (`Date.from(instant)`)
	- `of` - Aggregation (`EnumSet.of(JACK, QUEEN, KING)`)
	- `valueOf`
	- `instance`/`getInstance` - Returns instances described by parameters (if any) (`StackWalker luke = StackWalcker.getInstance(options)`)
	- `create` or `newInstance` - Guaranteed to create new instance
	- `getType`/`newType`/`type` - `Files.getFileStorePath`, `Files.newBufferedReader`,`Collections.list`
		- Like `getInstance`/`newInstance` but used if the factory method is in a different class
# Item 2: Consider a builder when faced with many constructor parameters
- Static factories and constructors do not scale well
	- Think of a `NutrificationFacts` class which has many optional parameters 
- **Telescoping constructor**: 
	- Hard to read/write
	- Have to pass in optional parameters
	- Often originate as a workaround for when classes expand beyond initial definition of required parameters
	```java
NutritionFacts cola = new NutritionFacts(240, 8); // This constructor in turn calls others with optional values, hence "telescoping"
NutritionFacts cola = new NutritionFacts(240, 8, 0, 100, 0, ...);
```
- **JavaBeans pattern**: 
	- Possible inconsistent state partway through construction
	- Class cannot be immutable
```java
NutritionFacts cola = new NutritionFacts();
cola.setServingSize(240);
cola.setServings(8);
...
```
- **Builder pattern**
```java
NutritionFacts nutritionFacts = new NutritionFacts.Builder(240, 8).calories(200) .fat(5).sodium(300).carbohydrate(27).build();
```

### Advantages
1. **Simulates optional parameters**
	- Optional parameters can be initialized with default values
	- Non-optional ones can go in constructor
2. **Validity checks as soon possible when setting parameters**
	- Throw `IllegalArgumentException` if checks fail
3. **Well suited to class hierarchies**
```java
// Each class below subclasses an abstract Pizza class which contains a nested builder class
NyPizza nyPizza = new NyPizza.Builder(NyPizza.Size.LARGE) .addTopping(Pizza.Topping.SAUSAGE).addTopping(Pizza.Topping.ONION).build(); Calzone calzone = new Calzone.Builder().addTopping(Pizza.Topping.HAM).sauceInside().build();
```

### Limitations
1. Enough parameters to make it worth it (say four or more)
2. Builder must be created first before the object, *may* be problematic in performance critical situations
3. Compile time vs runtime checking - this is why it's encouraged to put required variables in the Builder constructor
- When using a constructor, all your required parameters are checked at compile time
- With a builder, only the ones to the Builder are checked, everything else is a runtime check

# Item 3: Enforce the singleton property with a private constructor or an enum type
- **Singleton**: class that is instantiated exactly once

**Public final static member with private constructor**
- Simple
- Clear that it is a singleton from `final` usage
```java
public class Singleton { 
	public static final Singleton INSTANCE = new Singleton(); 
	private Singleton() {} 
}
```

**Public static factory method**
- Flexibility to change this to a non-singleton
- Can be made into a generic singleton factory
- Method reference can be used as a supplier (`Singleton::getInstance` is a `Supplier<Singleton>`)
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

Both of the above must define a `readResolve` method to support serialization, whereas the approach below does not.

**Enum**
- Often the best way to implement a singleton
- Serialization machinery for free
- However, cannot extend a superclass other than `Enum` (can implement interfaces though)
```java
public enum Singleton {
    INSTANCE;
}
```

# Item 4: Enforce noninstantiability with a private constructor
- Utility classes (groups of static methods and fields) were not designed to be instantiated, however the compiler provides a public and parameterless **default constructor**
- However, a default constructor is only generated if a class contains no explicit constructors, so a class can be made **noninstantiable** by including a private constructor
	- This makes it inaccessible outside the class
	- The `AssertionError` is thrown as a safeguard to indicate that instantiation isn’t allowed, even through reflection.
```java
public class UtilityClass {
    // Private constructor to prevent instantiation
    private UtilityClass() {
        throw new AssertionError("Cannot instantiate UtilityClass");
    }

    // Static utility methods
    public static void usefulMethod() {
        System.out.println("This is a utility method.");
    }
}
```

# Item 5: Enforce noninstantiability with a private constructor