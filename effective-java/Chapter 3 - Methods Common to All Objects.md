# Item 11: Always override hashCode when you override equals
- Failing to do so means that your class will violate the `hashCode` contract, which will prevent it from functioning properly in collections such as `HashMap` and `HashSet`
- The default `hashCode` implementation uses an[ Xorshift algorithm](https://stackoverflow.com/questions/49172698/default-hashcode-implementation-for-java-objects) to generate a random number
- Note: **significant fields** are those used in `equals` comparison
### Contract (`hashCode`)
Source: [Object Specification](https://hg.openjdk.org/jdk8u/jdk8u/jdk/file/a71d26266469/src/share/classes/java/lang/Object.java#l65)
1. Must return the same value consistently provided that no information used in `equals` comparison is modified
2. Two equal objects will have the same integer `hashCode`
3. Two unequal objects *do not* necessarily need to have distinct `hashCode` values, however distinct results may improve the performance of hash tables
	- Want to avoid non-uniform distributions across buckets
- The second provision is violated if you don't override `hashCode` after overriding `equals`

```Java

HashMap<PhoneNumber, String> phoneBook = new HashMap<>();
PhoneNumber p1 = new PhoneNumber(123, 456, 7890);
PhoneNumber p2 = new PhoneNumber(123, 456, 7890);
phoneBook.put(p1, "Alice");
        
// This will only work if hashCode is overriden
phoneBook.get(p2)); // Alice
```

### Guide to writing `hashCode`
1. Declare an `int` variable `result` initialized to your first significant field
2. Compute an `int` hash code `c` for each other significant field
	- Use `Type.hashCode(f)` for primitive types, where `Type` is the boxed primitive class
	- Recursively call `hashCode` for objects, use 0 for null
	- For arrays treat each significant element as a separate field, if all are significant use `Arrays.hashCode`
3. Combine using `result = 31 * result + c`
	- 31 is odd, if it was even and multiplication overflowed, information would be lost (multiplying by 2 is equivalent to shifting, whereas for odd numbers the least significant bit is maintained)
	- 31 is prime, prime numbers generally help reduce collisions
	- An optimization of 31 is that it can be replaced by a shift and subtraction `31 * i == (i << 5) - i`
	- The multiplication makes the result depend on **order of the fields**, otherwise for example, anagram `String` objects would have the same hash code
```Java
import java.util.Objects;

public final class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNumber;

    @Override
    public int hashCode() {
        // Example hashCode implementation
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(prefix);
        result = 31 * result + Integer.hashCode(lineNumber);
        return result;
    }
}
```
- Consider caching hash code if computation is significant, either lazy initialized or right when instance is created
- You should usually use **all significant fields** to maintain good hash code distribution

- `Objects` class provides a a static method for `hashCode` (`Objects.hash()`) but this has worse performance due to array creation for variable arguments and boxing/unboxing of primitives, recommended if performance isn't critical
