Effective Java - 3rd edition

# Fundamental Principles
- Clarity and simplicity
- Components should be as small as possible but no smaller
- Code should be reused rather than copied
- Dependencies between two components should be kept to a minimum
- Errors should be detected as soon as possible, ideally at compile time

# Java 
- The book uses technical terms defined in Java Standard Edition (SE) 8
- This language supports four kinds of types, with the first three being **reference types**:
	- Interfaces (including annotations)
	- Classes (including enums)
	- Arrays
	- Primitives

# Terminology
This terminology is not always specification to the Java Language Specification
- **Method signature**: a method's name and the types of parameters
	- Does *not* include return type
- **Subclassing**: used synonymously with inheritance
- **Package-private**: The default access level when none are specified, e.g. `class MyClass`
	- The class, method or variable is only accessible within the same package
	- However they are not accessible to subclasses (this differentiates it from **protected**)
- **Exported API**: The classes, interfaces, constructors members and serialized forms which which a class, interface or package can be accessed 
