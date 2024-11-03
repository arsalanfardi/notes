# Item 1: Consider static factory methods instead of constructors
- **Static factory method**: A static method that returns an instance of the class
	- Not to be confused with Factory design pattern
	- Below is an example from the *boxed primitive* class for boolean (Boolean)
```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
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
