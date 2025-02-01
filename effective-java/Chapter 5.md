- Generics are a specifically a compile time construct, they give us compile time type safety
- Java works with **type erasure,** meaning at runtime, your `List<Integer>` for example is just `List`

# Item 29: Favour generic types
- We can *generify* a non-generic Stack implementation without harming the original non-parameterized (`Object` type) version 
	- This provides type safety at compile time, different type elements can no longer be pushed into the same `Stack`

```Java
// Initial attempt, won't compile!
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    @SuppressWarnings("unchecked")
    public Stack() {
        // Creating a generic array is not allowed directly
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
}
```

- You can't create an array with a generic (non-reifiable) type
	- **Reifiable** types are types whose type information is fully available at runtime, such as primitives, non-generic types like `String` or `Integer`, or raw types like `List`
	- **Non-reifiable** types lose their type information due to type erasure

**Solutions**
1. Cast an array of object to generic array type. We can suppress the "unchecked" warning because we know the pushed type is always `E` and `elements` is a private field.
```Java
@SuppressWarnings("unchecked") 
public Stack() { 
	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```
2. Convert `elements` from `E[]` to `Object`. This will require casting the element each time we retrieve an item from the array, however we can again suppress the warning.
```Java
// Casting is required since elements is of type Object[]
@SuppressWarnings("unchecked") E result = (E) elements[--size]
```

- The first solution is more readable, concise and prevents the need for a separate cast each time. However it can result in **heap pollution** where runtime type does not match compile-time type (`Object`).
- Realistically could use `List<E>` instead and avoid this problem altogether
# Item 30: Favour generic methods
- Generic methods allow for:
	- Type safety at compile time (prevent `ClassCastException`)
	- Reusable code
	- Avoiding casting (which would the case prior to generics with only raw types)

Note: In the `<E>` before `Set<E>` in the method's signature is a placeholder for a type which will be called at runtime. This is what makes the method generic.
```Java
import java.util.HashSet;
import java.util.Set;

public class GenericMethodsExample {
    // Generic method to compute the union of two sets
    public static <E> Set<E> union(Set<E> set1, Set<E> set2) {
        Set<E> result = new HashSet<>(set1);
        result.addAll(set2);
        return result;
    }

    public static void main(String[] args) {
        Set<String> set1 = Set.of("Apple", "Banana", "Cherry");
        Set<String> set2 = Set.of("Date", "Elderberry", "Fig", "Cherry");

        // Using the generic method
        Set<String> resultSet = union(set1, set2);
    }
}
```

- **Generic singleton factory**: A single reusable instance of an object for any type while maintaining type safety.

Note: 
- An identity function is a function which always returns the value of its argument unchanged
- A `UnaryOperator` in Java is a functional interface from the `java.util.function` package, introduced in Java 8. It represents a function that takes a single input of a specific type and produces a result of the same type.
```Java
import java.util.function.UnaryOperator;

public class IdentityFunctionDispenser {
    // A reusable instance of the identity function for all types
    private static final UnaryOperator<Object> IDENTITY_FN = t -> t;

    // Generic method that returns the identity function for any type
    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }
}
```

- **Recursive type bound**: A type parameter bounded by a generic type that involves the type parameter itself, e.g.  `<T extends Comparable<T>>`

The example below ensures *mutual comparability*, which in practice is almost always the case (i.e. `String` implements `Comparable<String>`) but this would enforce it
- Here's a good [Stack Overflow discussion](https://stackoverflow.com/questions/7385949/what-does-recursive-type-bound-in-generics-mean#:~:text=Definition%20of%20a%20Recursive%20Type%20Bound&text=In%20our%20example%2C%20the%20generic,type%20bound%20Fruit%20.) on this
```Java
public static <T extends Comparable<T>> T max(Collection<T> c) {
    if (c.isEmpty()) {
        throw new IllegalArgumentException("Collection is empty");
    }
    
    T result = null;
    for (T element : c) {
        if (result == null || element.compareTo(result) > 0) {
            result = element;
        }
    }
    return result;
}
```