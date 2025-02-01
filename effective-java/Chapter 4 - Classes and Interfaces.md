# Item 15: Minimize the accessibility of classes and members
- A well designed component hides all its implementation details - **encapsulation** (or information hiding)
	- *Decouples* the components allowing for development in isolation
- Should strive to **make each class or member as inaccessible as possible**

### Classes:
- *package-private* (default when no access modifier is specified)- class is part of the implementation rather than the API
	- If the class is package-private and only used by one other class, consider making it a static nested class instead
- *public* - part of the API, obligated to support forever

### Members (fields, methods, nested classes/interfaces):
- *private* - Only accessible from top-level class where it is declared
- *package-private* - Accessible from any class in the package
- *protected* - Accessible from the package *and* from subclasses
- *public* - Accessible from anywhere

- Reflex should be to make all members not part of your API as private unless absolutely necessary
- For public classes, *protected* members is part of the classes public API and must be supported forever
- If a method overrides a super class, it cannot have a more restrictive access level in the subclass than in the super class
- **Don't** make a class, interface or member part of an exported API to facilitate testing
	- private -> package-private for a public class may be ok, but not any further than that
- Instance fields should **rarely** be public, especially if it's nonfinal or a reference to a **mutable** object
	- They are generally not thread-safe
	- Note that nonzero-length arrays are **always mutable**, so they should never be exposed - consider exposing via a getter returning an unmodifiable list or a copy instead

### Modules
- The *module system* was added in Java 9, *modules* are groupings of packages
- A module may explicitly export some of its packages 

# Item 16: In public classes use accessor methods not public fields
- Classes with public instance fields do not offer the benefits of *encapsulation*, you can't change the representation without changing the API
- For classes accessible outside of its package, provide **accessor methods** (getters) to maintain flexibility of changing internal representation
- However, if a class is *package-private* or a *private* nested class, data fields can be exposed 

# Item 17: Minimize mutability
- An immutable class is one where the instances cannot be modified (i.e. all of the information contained in each instance is fixed for the lifetime of the object)
	- Java offers many immutable classes like `String`, boxed primitives, `BigInteger`/`BigDecimal`
### How to make a class immutable
1. Don't provide mutator methods
2. Prevent subclassing
	- Prevents careless or malicious subclasses from bypassing immutability
	-  Do this by making the class final or making the constructor private
```Java
public class ImmutableParent {
    private final String value;

    public ImmutableParent(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}

public class MutableChild extends ImmutableParent {
    private String mutableValue;

    public MutableChild(String value, String mutableValue) {
        super(value);
        this.mutableValue = mutableValue;
    }

    // Add a mutable field with a setter
    public void setMutableValue(String mutableValue) {
        this.mutableValue = mutableValue;
    }

    @Override
    public String getValue() {
        // Simulates mutability of parent class
        return mutableValue;
    }
}
```
3. Make all fields final
4. Make all fields private
	- Prevents clients from accessing references to mutable objects
5. Ensure exclusive access to any mutable components
	- For any fields that refer to mutable objects, ensure clients cannot access them and that they are not directly returned from an accessor. Make defensive copies where necessary.

- Here's an example of an immutable class:
```Java
public final Complex {
	public Complex(double real, double imaginary) {
	    this.real = real;
	    this.imaginary = imaginary;
	}

	public Complex plus(Complex c) {
	    return new Complex(this.real + c.real, this.imaginary + c.imaginary);
	}
}
```
 - Rather than modifying the instance a new `Complex` instance is returned each time
 - This is known as a *functional* approach because methods return the result of the operand without modifying it
	 - As opposed to *procedure* or *imperative* which alter state

### Benefits of immutable classes
- Simplicity - immutable objects have exactly one state permanently
- Immutable objects are inherently thread-safe and require no synchronization
- Can be shared freely - should therefore reuse existing instances whenever possible, possibly through public static final constants or static factories
	- Defensive copies aren't required since all copies are always equal to the original - should not provide `clone` methods or copy constructors
```Java
public static final Complex ZERO = new Complex(0, 0);
```
- Immutable objects are always atomic, so there is no possibility of temporary inconsistency
- Great building blocks for other objects - e.g. can serve as map keys since you don't have to worry about values changing 

- The major **disadvantage** is that they require separate objects for each distinct value
	- Consider providing a companion class (package-private or public, like `StringBuilder` for `String`)

### Preventing subclassing
- Using a private or package-private constructor is more flexible than making a class final because:
	- Classes in the same package (package-private) or nested inner classes (private) can still extend the class
	- It is still effectively final to classes outside of its package
- Instead use public static factory methods

### Summary
- Technically no method in an immutable class can produce an *externally* visible change in the object's state, but it can use nonfinal fields to aid with things like caching expensive computations
- Classes should be immutable unless there's a very good reason otherwise
	- Particularly for small value classes (`PhoneNumber` or `Complex`)
	- Or limit immutability as much as possible
- Relating to item 15, declare every field private final unless there's a good reason otherwise
- Constructors should create fully initialized objects with invariants established

# Item 18: Favor composition over inheritance
- Inheritance is generally safe to use when in the same package and under control of the same programmers, however it can become problematic with public classes
- Inheritance **violates encapsulation**, but method invocation does
- Subclass depends on superclass implementation
	- Changes to superclass may break the subclass, despite the fact subclass has no direct changes
	- Needlessly exposes implementation details
- **Composition**: Superclass becomes a *component* of the new class
	- **Forwarding**: your new class can invoke the "superclass" methods (AKA *Wrapper* or *Decorator* design pattern)
	- Also relates to dependency injection

- Inheritance should only be used when "**is-a**" relation holds true (B is an A)
	- Otherwise, B is merely an implementation detail of A
- In summary: inheritance shouldn't necessarily be your default choice for code re-use

# Item 19: Design and document for inheritance or else prohibit it
- Don't call an overridable method from your constructor - can result in `NullPointerExceptions`

# Item 20: Prefer interfaces to abstract classes
- Classes can only extend one abstract class but multiple interfaces
```Java
public interface Singer {
    void sing();
}

public interface Songwriter {
    void compose(String songTitle);
}

public interface SingerSongwriter extends Singer, Songwriter {
    void perform(String songTitle);
}
```
- Interfaces are ideal for **mixins**, i.e. types that a class can implement in addition to primary type
	- Optional functionality is "mixed-in" to the type's primary functionality
	- Think `Comparable` interface
- Since Java 8, *default* methods were introduced for interfaces which allow you to provide implementations for instance methods
# Item 21: Design interfaces for posterity
- Need to be careful to not break existing implementation if adding a default method 

# Item 22: Use interfaces only to define types
- Don't use interfaces to define constants, interfaces should represent a contract or a type that classes implement, not a collection of constants
# Item 23: Prefer class hierarchies to tagged classes
- A **tagged** class contains a field that indicates its type subtype
# Item 24: Favour static member classes over nonstatic

# Item 25: Limit source files to a single top-level class
