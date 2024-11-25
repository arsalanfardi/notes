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
1. Make all fields final
2. Make all fields private
	- Prevents clients from accessing references to mutable objects
3. Ensure exclusive access to any mutable components
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


