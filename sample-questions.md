# 1Z0-831 Sample Questions with Answers

20 worked practice questions for the **Oracle Certified Professional: Java SE 25 Developer (1Z0-831)** exam. Try each one before you open the answer.

For many more, with the same explanation on every option, use the [1Z0-831 practice questions](https://www.savemycert.com/practice/oracle-java-se-developer/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide) on SaveMyCert.

## Question 1

*Handling Date, Time, Text, Numeric and Boolean Values*

**A method declares two local variables:
```java
byte a = 10;
byte b = 20;
```
Which statement, added after them, compiles?** Choose one.

- **a.** byte c = (byte) (a + b);
- **b.** byte c = a + (byte) b;
- **c.** byte c = (byte) a + (byte) b;
- **d.** byte c = (byte) a + b;

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The parentheses make the cast apply to the whole int sum, so the result is narrowed explicitly and the assignment compiles (it prints 30).
- **b** ❌ Casting b to byte changes nothing: byte + byte is still promoted to int, and an int expression that is not a constant cannot be assigned to a byte.
- **c** ❌ Casting each operand still leaves the addition itself promoted to int, so the sum is an int and the assignment fails to compile.
- **d** ❌ The cast applies to a alone, then a + b is promoted to int again, so assigning the int sum to a byte fails with 'possible lossy conversion from int to byte'.

**The concept.** Binary numeric promotion turns byte, short and char operands into int before any arithmetic operator runs.

**Why this is correct.** a + b has type int even though both operands are bytes, and b and a are not constants, so the sum needs an explicit cast. A cast binds tighter than +, so it only narrows the whole sum when the sum is in parentheses. Every other form casts an operand, which is then promoted back to int by the addition.

**How to reason it out**

1. Note that b and a are bytes but not final constants.
2. Apply promotion: byte + byte produces an int.
3. Check what each cast actually covers; a cast binds tighter than +.
4. Only a cast around the whole sum turns the int back into a byte.

> **Exam tip:** byte + byte is an int; cast the whole expression, not the operands.

</details>

📖 Learn this topic: [Java Primitives, Wrapper Classes, Operators and Casting (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/ocp-java-primitives-wrappers-operators-casting/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 2

*Handling Date, Time, Text, Numeric and Boolean Values*

**Given:
```java
public class Main {
    public static void main(String[] args) {
        String s = "java";
        s.toUpperCase();
        s.concat(" 25");
        System.out.println(s);
    }
}
```
What is the result?** Choose one.

- **a.** JAVA 25
- **b.** java
- **c.** JAVA
- **d.** java 25

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ This assumes both calls mutated s. Neither result is assigned back, so both new Strings are lost.
- **b** ✅ Neither toUpperCase() nor concat() changes s; each returns a new String that is discarded, so s still refers to "java".
- **c** ❌ This assumes toUpperCase() changed s in place. Strings are immutable; the uppercase copy is never assigned.
- **d** ❌ This assumes concat() appends to the existing object. It returns a new String, which is ignored here.

**The concept.** String objects are immutable: every "modifying" method returns a new String.

**Why this is correct.** The two method calls build new String objects but their return values are never stored. The variable s keeps pointing at the original literal, so println prints java. To keep a change you must reassign: s = s.toUpperCase().

**How to reason it out**

1. Find where s is assigned: only once, to "java".
2. Check whether any method result is assigned back to s: none is.
3. Remember that String methods never change the object they are called on.
4. The printed value is the original text.

> **Exam tip:** A String method result that is not assigned is simply lost.

</details>

📖 Learn this topic: [Java Strings, StringBuilder and Text Blocks (OCP Java SE 25)](https://www.savemycert.com/revision/oracle-java-se-developer/java-string-stringbuilder-text-blocks/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 3

*Implementing Program Flow Control Using Decision and Looping Constructs*

**Given:
```java
public class Main {
    public static void main(String[] args) {
        int a = 5, b = 10;
        if (a > 3)
            if (b < 5)
                System.out.print("X");
        else
            System.out.print("Y");
        System.out.print("Z");
    }
}
```
What is the result?** Choose one.

- **a.** YZ
- **b.** Compilation fails
- **c.** Z
- **d.** XZ

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The else binds to the nearest unmatched if, the inner if (b < 5). a > 3 is true and b < 5 is false, so the inner else prints Y, then Z is printed.
- **b** ❌ Nested if statements without braces are legal; the else simply attaches to the closest if that has no else yet.
- **c** ❌ This is what you get if the else is read as belonging to the outer if, as the indentation suggests. Java ignores indentation: the else pairs with the inner if.
- **d** ❌ X prints only when b < 5, and b is 10, so the inner condition is false.

**The concept.** Without braces, an else always pairs with the closest preceding if that does not already have an else (the dangling-else rule). Indentation has no meaning to the compiler.

**Why this is correct.** The code is laid out to suggest the else belongs to if (a > 3). The compiler pairs it with if (b < 5) instead. a > 3 is true, so the inner if runs; b < 5 is false, so its else prints Y. The final print is outside both ifs and prints Z, giving YZ. Reading the else as the outer one's gives Z, which is the trap.

**How to reason it out**

1. Ignore indentation and match each else to the nearest if without an else.
2. Evaluate the outer condition a > 3: true, so the inner if statement runs.
3. Evaluate b < 5: false, so the inner else prints Y.
4. The last print is unconditional and prints Z.

> **Exam tip:** An else belongs to the nearest unmatched if; use braces when you mean otherwise.

</details>

📖 Learn this topic: [Java Flow Control: if/else, switch and Loops (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-flow-control-if-switch-loops-break-continue/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 4

*Implementing Program Flow Control Using Decision and Looping Constructs*

**A developer is choosing the if header for line X in this class:
```java
public class Main {
    public static void main(String[] args) {
        int count = 3;
        boolean ready = false;
        // line X
            System.out.print("go");
    }
}
```
Which two if headers, placed at line X, compile? (Select TWO.)** Choose 2.

- **a.** if (ready = count > 0)
- **b.** if (count = count - 3)
- **c.** if (Integer.valueOf(count))
- **d.** if (Boolean.valueOf(ready))
- **e.** if (ready | Integer.signum(count))

<details>
<summary>Show answer and explanation</summary>

**Answer: a, d**

- **a** ✅ count > 0 is boolean, so assigning it to ready is a boolean expression. It compiles, sets ready to true and prints go.
- **b** ❌ count = count - 3 is an int assignment, so the condition has type int (even though its value would be 0). The compiler reports that int cannot be converted to boolean.
- **c** ❌ Integer unboxes to int, not boolean. Java has no number-to-boolean conversion, so javac reports that Integer cannot be converted to boolean.
- **d** ✅ Boolean.valueOf returns a Boolean, which is unboxed to boolean, so it is a valid condition. With ready false it prints nothing, but it compiles.
- **e** ❌ Integer.signum returns an int. The | operator needs two boolean or two integral operands, so mixing boolean and int fails with bad operand types.

**The concept.** An if condition must have type boolean or Boolean (unboxed). Java never converts numbers to boolean, so int expressions and int assignments are not conditions.

**Why this is correct.** Assigning a boolean inside the condition is legal because the assignment expression has the boolean type of its left side: ready = count > 0 is boolean. A Boolean object is unboxed automatically. Assigning an int, using an Integer, or mixing boolean with int in | all produce compile errors, because there is no numeric truthiness in Java.

**How to reason it out**

1. Work out the static type of each condition expression.
2. An assignment expression has the type of its left-hand variable.
3. Accept boolean and Boolean (auto-unboxed); reject int.
4. Check that binary operators such as | get matching operand types.

> **Exam tip:** Conditions must be boolean or Boolean; Java has no int truthiness, and x = 0 in an if does not compile.

</details>

📖 Learn this topic: [Java Flow Control: if/else, switch and Loops (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-flow-control-if-switch-loops-break-continue/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 5

*Applying Object-Oriented Principles in Java Programs*

**Given:
```java
class Engine {
    void start() {
        System.out.println("started");
    }
}

class Car {
    Engine engine;
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.engine.start();
    }
}
```
What is the result?** Choose one.

- **a.** Nothing is printed and the program ends normally
- **b.** Compilation fails
- **c.** started
- **d.** A NullPointerException is thrown

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Calling an instance method through a null reference is not silently skipped; the JVM throws.
- **b** ❌ The code is legal: a field of a reference type may be declared without an initializer, and calling a method through it compiles.
- **c** ❌ Declaring the field engine does not create an Engine object; no object exists to run start().
- **d** ✅ The engine field was never assigned, so it keeps its default value null and calling start() through it throws.

**The concept.** Declaring a reference creates no object. Only new (or another factory that calls new) creates one; an unassigned reference field defaults to null.

**Why this is correct.** new Car() creates one Car object whose engine field gets the default value for references, null. No Engine is ever created. car.engine.start() therefore dereferences null and the JVM throws a NullPointerException (Java's helpful message names engine as the null). The code compiles because the compiler does not track whether a field has been assigned; only local variables must be definitely assigned before use.

**How to reason it out**

1. List which objects new actually creates: one Car.
2. Recall the default value of an object reference field: null.
3. Calling a method on null throws NullPointerException at runtime.
4. Fields are not checked for definite assignment, so it compiles.

> **Exam tip:** A reference variable is not an object: until something is assigned, a reference field is null.

</details>

📖 Learn this topic: [Java Objects, Nested Classes and Garbage Collection (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-objects-nested-classes-garbage-collection/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 6

*Applying Object-Oriented Principles in Java Programs*

**Given:
```java
class Box {
    int size;

    Box(int size) {
        this.size = size;
    }
}

public class Main {
    public static void main(String[] args) {
        Box b = new Box();
        System.out.println(b.size);
    }
}
```
What is the result?** Choose one.

- **a.** The program prints nothing
- **b.** An exception is thrown at runtime
- **c.** Compilation fails
- **d.** 0

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ A program that fails to compile does not run at all; this would need main to run and skip the print.
- **b** ❌ The problem is found by the compiler; the program never starts, so no runtime exception can occur.
- **c** ✅ Box declares a constructor taking an int, so the compiler adds no no-argument constructor and new Box() has nothing to call.
- **d** ❌ 0 would be the default value of size, but no Box object can be created this way, so nothing runs.

**The concept.** The compiler adds a default no-argument constructor only when a class declares no constructors at all.

**Why this is correct.** Box declares Box(int size). Because at least one constructor is written, the compiler does not generate the default no-argument constructor. The call new Box() therefore matches no constructor and javac reports "constructor Box in class Box cannot be applied to given types; required: int". The 0 distractor assumes a default constructor still exists alongside the written one.

**How to reason it out**

1. Count the constructors Box declares: one, taking an int.
2. Recall that a default constructor is generated only when none are declared.
3. Match new Box() against the declared constructors: no match.
4. A failed constructor match is a compile-time error.

> **Exam tip:** Write any constructor and the free no-argument one disappears; add Box() yourself if callers need it.

</details>

📖 Learn this topic: [Java Classes, Records, Constructors and Initializers (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-classes-records-constructors-initializers/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 7

*Applying Object-Oriented Principles in Java Programs*

**A class Calc already declares this method:
```java
int area(int w, int h) { return w * h; }
```
Which declaration, added to the same class, causes compilation to fail?** Choose one.

- **a.** int area(int side) { return side * side; }
- **b.** long area(int a, int b) { return a * b; }
- **c.** int area(long w, int h) { return (int) w * h; }
- **d.** double area(double w, double h) { return w * h; }

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ A different number of parameters is a different signature, so this overload compiles.
- **b** ✅ Its parameter types are (int, int), the same as the existing method; changing the return type and the parameter names does not create a new signature, so javac reports that area(int,int) is already defined.
- **c** ❌ The first parameter type differs (long instead of int), so this is a legal overload.
- **d** ❌ The parameter types (double, double) differ, so this is a legal overload; the different return type is irrelevant.

**The concept.** A method's signature is its name plus the number, types and order of its parameters. Overloads must differ in that list; return type and parameter names are not part of it.

**Why this is correct.** Overloading is decided by the parameter type list only. The added method long area(int a, int b) has the parameter types (int, int), exactly like int area(int w, int h), so the two methods have the same signature and the compiler rejects the class with 'method area(int,int) is already defined'. Renaming the parameters and changing the return type do not help. The other three declarations change the number or the types of the parameters, so each is a valid overload.

**How to reason it out**

1. Write down each method's parameter types in order, ignoring names.
2. Compare them with (int, int) from the existing method.
3. Ignore the return type: it never distinguishes overloads.
4. The declaration whose list is still (int, int) is a duplicate.

> **Exam tip:** Overloads differ in parameter types, count or order; a different return type alone is a duplicate method.

</details>

📖 Learn this topic: [Java Method Overloading and Varargs Rules (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-method-overloading-varargs/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 8

*Applying Object-Oriented Principles in Java Programs*

**Given:
```java
public class Main {
    static int total;

    public static void main(String[] args) {
        int count;
        System.out.println(total + count);
    }
}
```
What is the result?** Choose one.

- **a.** A NullPointerException is thrown
- **b.** Compilation fails
- **c.** An unpredictable number is printed
- **d.** 0

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Both variables are primitives, so there is no reference to be null; the problem is caught by the compiler, not at run time.
- **b** ✅ count is a local variable that is read before anything is assigned to it; javac reports 'variable count might not have been initialized'.
- **c** ❌ Java never exposes uninitialised memory: a local must be definitely assigned before it is read, or the code does not compile.
- **d** ❌ total would print 0 as a field default, but count is a local variable and locals get no default value, so the code never runs.

**The concept.** Fields (static and instance) get default values; local variables do not and must be definitely assigned before they are read.

**Why this is correct.** The static field total is initialised to 0 automatically. The local variable count is declared without an initializer and is then read in the println call. The compiler's definite-assignment analysis finds no assignment on that path and rejects the program with 'variable count might not have been initialized'. Java never prints garbage values and never defers this check to run time.

**How to reason it out**

1. Classify each variable: total is a static field, count is a local.
2. Fields get defaults (0 for int); locals get none.
3. count is read before any assignment, so definite assignment fails.
4. The result is a compile error, not output or an exception.

> **Exam tip:** Only fields get default values; reading an unassigned local is a compile error.

</details>

📖 Learn this topic: [Java Variable Scope, Encapsulation, Immutability, var and _ (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-variable-scope-encapsulation-immutability-var/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 9

*Implementing Exception Handling in Java Applications*

**Given:
```java
public class Main {
    static void load(String id) {
        if (id.isBlank()) {
            throw new Exception("no id");
        }
        System.out.println("loaded " + id);
    }

    public static void main(String[] args) {
        load("42");
    }
}
```
What is the result?** Choose one.

- **a.** An Exception is thrown at runtime
- **b.** Compilation fails
- **c.** Nothing is printed
- **d.** loaded 42

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ The throw would only run for a blank id, and it never gets the chance: the checked-exception rule is enforced at compile time, not at runtime.
- **b** ✅ new Exception(...) is a checked exception. load() has no catch for it and no throws clause, so javac reports: unreported exception Exception; must be caught or declared to be thrown.
- **c** ❌ Silence would need the program to compile and skip the print. It does not compile at all.
- **d** ❌ The program never runs. javac rejects load() first, because it throws a checked exception it neither catches nor declares.

**The concept.** A checked exception (Exception and its subclasses other than RuntimeException) must be caught or declared wherever it can be thrown.

**Why this is correct.** The compiler checks every throw statement and call for checked exceptions. Exception itself is checked, so the throw inside load() needs either a surrounding catch or a throws Exception clause on load(). Neither exists, so compilation fails, even though the branch would not run for the id 42. The runtime-exception answer assumes Java checks this only when the throw executes; it does not.

**How to reason it out**

1. Classify the thrown type: Exception is checked.
2. Look for a catch in load() that handles it: none.
3. Look for a throws clause on load(): none.
4. Conclude that javac rejects the method, whatever the argument.

> **Exam tip:** Checked exceptions are a compile-time rule: catch them or declare them, even on branches that never run.

</details>

📖 Learn this topic: [Java Exception Handling: try/catch/finally, try-with-resources and Custom Exceptions (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-exceptions-try-catch-finally-try-with-resources/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 10

*Implementing Exception Handling in Java Applications*

**Given:
```java
public class Main {
    public static void main(String[] args) {
        try {
            System.out.print("A");
            Integer.parseInt("x1");
            System.out.print("B");
        } catch (NumberFormatException e) {
            System.out.print("C");
        } finally {
            System.out.print("D");
        }
        System.out.print("E");
    }
}
```
What is the result?** Choose one.

- **a.** ACDE
- **b.** ADE
- **c.** ABCDE
- **d.** ACD

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ parseInt("x1") throws NumberFormatException, so B is skipped, the catch prints C, finally prints D, and execution continues after the statement to print E.
- **b** ❌ NumberFormatException matches the catch clause, so C is printed before the finally block runs.
- **c** ❌ B never prints: the exception leaves the try block at parseInt, and the rest of the try block is skipped.
- **d** ❌ The exception is handled by the catch, so the try statement completes normally and the statement after it, which prints E, runs.

**The concept.** When a try block throws, the rest of the block is skipped, a matching catch runs, then finally, and execution continues after the statement.

**Why this is correct.** A prints first. Integer.parseInt("x1") throws NumberFormatException, which abandons the remaining statements of the try block, so B is not printed. The matching catch prints C, the finally block always runs and prints D, and because the exception was handled the program carries on and prints E. The tempting ACD forgets that a handled exception lets the method continue normally.

**How to reason it out**

1. Run the try block until the first exception: A, then parseInt throws.
2. Skip the rest of the try block.
3. Run the matching catch, then the finally block.
4. Continue after the try statement because the exception was handled.

> **Exam tip:** A handled exception skips the rest of try, runs catch and finally, then carries on after the statement.

</details>

📖 Learn this topic: [Java Exception Handling: try/catch/finally, try-with-resources and Custom Exceptions (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-exceptions-try-catch-finally-try-with-resources/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 11

*Using Arrays and Collections to Store and Retrieve Data*

**Given:
```java
public class Main {
    public static void main(String[] args) {
        int[] counts = new int[3];
        String[] names = new String[2];
        boolean[] flags = new boolean[1];
        System.out.println(counts[2] + " " + names[1] + " "
            + flags[0] + " " + counts.length);
    }
}
```
What is the result?** Choose one.

- **a.** 0  false 2
- **b.** Compilation fails
- **c.** 0 null false 3
- **d.** 0 null false 2

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ A String element defaults to null, not to the empty string, and length is the element count 3, not the last index 2.
- **b** ❌ Only local variables must be assigned before use; array elements get default values, so reading counts[2] compiles.
- **c** ✅ Array elements are always initialised: 0 for int, null for references, false for boolean; length is the size given to new, 3.
- **d** ❌ counts was created with new int[3], so its length is 3; 2 is the last valid index, not the length.

**The concept.** Creating an array with new allocates every element and sets it to the type's default value: 0, 0.0, false, '\u0000' or null.

**Why this is correct.** new int[3] creates three int elements set to 0, new String[2] two references set to null, and new boolean[1] one false. Unlike local variables, array elements never need explicit initialisation, so the code compiles. String concatenation turns the null reference into the text null, and length reports the number of elements, 3.

**How to reason it out**

1. Note that each array is created with new and a size.
2. Apply the default value for each element type.
3. Remember that null concatenates as the text null.
4. Read length as the size, not the last index.

> **Exam tip:** Array elements default to 0, false or null; length is the element count.

</details>

📖 Learn this topic: [Java Arrays and Collections: List, Set, Map, Deque and Sequenced Collections (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-arrays-collections-list-set-map-deque/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 12

*Processing Data Using Streams and Lambda Expressions*

**Given:
```java
import java.util.stream.Stream;

public class Main {
    public static void main(String[] args) {
        Stream<String> s = Stream.of("a", "b", "c")
            .peek(x -> System.out.print(x))
            .map(String::toUpperCase);
        System.out.print("done");
    }
}
```
What is the result?** Choose one.

- **a.** ABCdone
- **b.** abcdone
- **c.** Compilation fails
- **d.** done

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Neither peek nor map runs, so nothing is printed before done; and peek would print the elements before map upper-cases them anyway.
- **b** ❌ This assumes peek runs as soon as the pipeline is built. Intermediate operations only run when a terminal operation pulls elements, and this pipeline has none.
- **c** ❌ Assigning an unfinished pipeline to a Stream<String> variable is legal; leaving it unused is not a compile error.
- **d** ✅ peek and map are intermediate operations. Without a terminal operation such as forEach or count, no element is ever pulled through them, so only done is printed.

**The concept.** Intermediate stream operations are lazy: they describe work and run only when a terminal operation consumes the stream.

**Why this is correct.** Stream.of, peek and map all return new Stream objects without touching any element. Elements are pulled only by a terminal operation (forEach, collect, count, findFirst and so on). The variable s is never given one, so the lambda inside peek never executes and the only output is done. The tempting answers abcdone and ABCdone both assume the pipeline runs eagerly.

**How to reason it out**

1. Classify each call: Stream.of creates, peek and map are intermediate.
2. Look for a terminal operation on s; there is none.
3. Conclude that no lambda in the pipeline runs.
4. Only the final print statement produces output.

> **Exam tip:** No terminal operation, no work: peek, map and filter never run on their own.

</details>

📖 Learn this topic: [Java Streams, Lambdas and Primitive Streams (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-streams-lambda-expressions-primitive-streams/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 13

*Processing Data Using Streams and Lambda Expressions*

**Given:
```java
import java.util.*;
import java.util.stream.*;

public class Main {
    public static void main(String[] args) {
        Map<Boolean, List<Integer>> m = Stream.of(2, 4, 6)
            .collect(Collectors.partitioningBy(n -> n > 10));
        System.out.println(m);
    }
}
```
What is the result?** Choose one.

- **a.** {false=[2, 4, 6], true=[]}
- **b.** {false=[], true=[2, 4, 6]}
- **c.** {true=[], false=[2, 4, 6]}
- **d.** {false=[2, 4, 6]}

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ partitioningBy always creates both keys. No element exceeds 10, so every element lands under false and the true list exists but is empty.
- **b** ❌ The predicate is n > 10, and none of 2, 4 or 6 is greater than 10, so they cannot be in the true list.
- **c** ❌ Both keys are present, but the map partitioningBy returns lists false before true, so this ordering is not what toString prints.
- **d** ❌ This is what groupingBy with a boolean classifier would print, since it only creates keys that occur. partitioningBy always holds both keys.

**The concept.** Collectors.partitioningBy splits a stream into exactly two groups keyed by false and true, and both keys are always present.

**Why this is correct.** The predicate n > 10 is false for every element, so 2, 4 and 6 all go to the false list. Unlike groupingBy, partitioningBy pre-populates both keys, so the true key maps to an empty list rather than being absent. The returned map prints false first, giving {false=[2, 4, 6], true=[]}.

**How to reason it out**

1. Evaluate the predicate for each element: all are false.
2. Remember that partitioningBy always creates both the false and true keys.
3. Note the empty list for the unused key.
4. Read the map in its printed order: false, then true.

> **Exam tip:** partitioningBy always returns both keys; an empty partition is an empty list, not a missing key.

</details>

📖 Learn this topic: [Java Stream Reduction, Grouping, Partitioning and Gatherers (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-stream-reduction-grouping-partitioning-gatherers/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 14

*Packaging and Deploying Java Code*

**Given:
```java
// lib/module-info.java
module lib {
    exports lib.api;
}

// lib/lib/api/Tax.java
package lib.api;

public class Tax {
    public static int rate() {
        return lib.internal.Table.base();
    }
}

// lib/lib/internal/Table.java
package lib.internal;

public class Table {
    public static int base() { return 20; }
}

// app/module-info.java
module app {
    requires lib;
}

// app/app/Main.java
package app;

public class Main {
    public static void main(String[] args) {
        int r = lib.api.Tax.rate();
        int b = lib.internal.Table.base();
        System.out.println(r + b);
    }
}
```
All modules are compiled together with javac --module-source-path, and the program is launched with java -p out -m app/app.Main. What is the result?** Choose one.

- **a.** It prints 40.
- **b.** It prints 20, then throws an exception.
- **c.** It throws IllegalAccessError.
- **d.** Compilation fails.

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Only lib.api is exported. Tax may call Table internally because both are in module lib, but app's direct reference to lib.internal.Table is rejected by the compiler, so nothing runs.
- **b** ❌ Nothing is printed: the reference to lib.internal.Table in Main is a compile-time error, so the program is never launched.
- **c** ❌ Module encapsulation is checked by javac first. Because the source refers to an unexported package, compilation stops; there is no class file left to fail at runtime.
- **d** ✅ lib exports only lib.api. The public class Table sits in lib.internal, which lib does not export, so javac reports that package lib.internal is declared in module lib, which does not export it.

**The concept.** A module's public types are accessible to other modules only if their package is exported; public on the class is not enough.

**Why this is correct.** Accessibility across modules needs three things: app reads lib (requires lib), the package is exported to app, and the type is public. Table is public, but lib.internal is not exported, so javac rejects lib.internal.Table in app. Code inside lib (Tax) can still use Table freely, which is why Tax compiles.

**How to reason it out**

1. List the packages lib exports: only lib.api.
2. Find every type app names directly: Tax and Table.
3. Table lives in lib.internal, which is not exported.
4. An access across a module boundary to an unexported package is a compile-time error.

> **Exam tip:** public means public to the module; only exported packages are visible to other modules.

</details>

📖 Learn this topic: [Java Modules, Services and Module Imports (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-modules-services-module-imports/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 15

*Packaging and Deploying Java Code*

**The file Main.java contains only the following code:
```java
int count;

void main() {
    count++;
    bump();
    IO.println(count);
}

void bump() {
    count += 10;
}
```
It is run with `java Main.java`. What is the result?** Choose one.

- **a.** 1
- **b.** Compilation fails
- **c.** 10
- **d.** 11

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ This assumes bump changes a different variable. count is a field of the implicit class, and bump runs on the same instance, so its += 10 is kept.
- **b** ❌ Top-level fields and methods with no class declaration are legal in Java SE 25: the file is a compact source file and compiles to an implicit class.
- **c** ❌ This drops the count++ that runs before bump. The field goes 0 to 1, then bump adds 10.
- **d** ✅ The top-level field and both methods belong to the one implicitly declared class. main and bump work on the same instance field: 0 + 1 + 10 = 11.

**The concept.** A compact source file's top-level fields and methods become members of one implicitly declared class.

**Why this is correct.** Because the file has fields and methods outside any class, the compiler wraps them in an implicitly declared final class named after the file. count is an ordinary instance field of that class. The launcher creates one instance and calls main on it; count++ makes it 1 and bump adds 10, so 11 is printed.

**How to reason it out**

1. Recognise the file as a compact source file: members with no enclosing class.
2. Treat count as an instance field of the implicit class, starting at 0.
3. Trace main: count++ gives 1, bump() adds 10.
4. IO.println prints 11.

> **Exam tip:** Code outside any class in a .java file is a compact source file; its members behave like members of one ordinary class.

</details>

📖 Learn this topic: [Compiling and Running Java: Compact Source Files, JARs, jlink and Module Migration (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-compile-run-jars-jlink-module-migration/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 16

*Implementing Multithreading for Concurrent Code Execution*

**Given:
```java
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(() -> System.out.print(
            Thread.currentThread().getName() + " "));
        t.setName("worker");
        t.run();
        System.out.print(Thread.currentThread().getName());
    }
}
```
What is the result?** Choose one.

- **a.** Thread-0 main
- **b.** worker main
- **c.** main main
- **d.** main worker

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Thread-0 is the default name of the Thread object, but setName replaced it, and the lambda does not run on that thread anyway.
- **b** ❌ This would be the output of t.start(), where the lambda runs on the new thread named worker. Calling run() never creates a thread.
- **c** ✅ run() is an ordinary method call, so the lambda executes on the calling thread, which is main, and then main prints its own name.
- **d** ❌ The order of the prints is fixed by the code: the lambda's print happens first, and both prints report the thread that executed them.

**The concept.** Thread.start() asks the JVM to run the task on a new thread; calling run() directly just executes the Runnable on the current thread.

**Why this is correct.** t.run() is a plain method invocation. Thread.run() calls the Runnable passed to the constructor, but on whichever thread called run(), here main. Thread.currentThread() therefore returns the main thread both times, and the name set on t is never observed. The program prints main main.

**How to reason it out**

1. Notice that the code calls run(), not start().
2. A direct run() call executes the task on the calling thread.
3. The calling thread is main, so the lambda prints main.
4. The last statement prints main again.

> **Exam tip:** Only start() creates a new thread of execution; run() is just a method call.

</details>

📖 Learn this topic: [Java Threads, Virtual Threads, Executors and Scoped Values (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-virtual-threads-executors-callable-scoped-values/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 17

*Implementing Multithreading for Concurrent Code Execution*

**Given:
```java
import java.util.concurrent.atomic.AtomicInteger;

public class Main {
    public static void main(String[] args) {
        var hits = new AtomicInteger(5);
        int first = hits.getAndIncrement();
        int second = hits.incrementAndGet();
        System.out.println(first + " " + second + " " + hits.get());
    }
}
```
What is the result?** Choose one.

- **a.** 5 6 6
- **b.** 5 7 7
- **c.** 6 6 7
- **d.** 6 7 7

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Both calls increment the counter. After two increments from 5 the counter holds 7, so second and get() are 7.
- **b** ✅ getAndIncrement returns 5 and moves the counter to 6; incrementAndGet moves it to 7 and returns 7; get() reads 7.
- **c** ❌ This swaps the two return rules. getAndIncrement returns the old value and incrementAndGet returns the new one.
- **d** ❌ getAndIncrement returns the value before the increment, so first is 5, not 6.

**The concept.** AtomicInteger names its methods by order: getAndX returns the value before the change, xAndGet returns the value after it.

**Why this is correct.** hits starts at 5. getAndIncrement() returns 5 and leaves 6. incrementAndGet() changes 6 to 7 and returns 7. get() then reads 7, so the line is 5 7 7.

**How to reason it out**

1. Track the stored value separately from each return value.
2. getAndIncrement: return 5, store 6.
3. incrementAndGet: store 7, return 7.
4. get() reads the stored 7.

> **Exam tip:** getAndX returns the old value; xAndGet returns the new one.

</details>

📖 Learn this topic: [Java Thread Safety: synchronized, Locks, volatile and Atomic Variables (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-thread-safety-synchronized-locks-atomic-variables/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 18

*Performing Input and Output Operations Using the Java I/O API*

**A developer is sorting java.io classes into byte streams and character streams. Which two classes are character streams? (Select TWO.)** Choose 2.

- **a.** DataInputStream
- **b.** PrintStream
- **c.** BufferedWriter
- **d.** FileReader
- **e.** FileInputStream

<details>
<summary>Show answer and explanation</summary>

**Answer: c, d**

- **a** ❌ DataInputStream extends FilterInputStream (an InputStream). It reads primitive values from bytes, so it is a byte stream.
- **b** ❌ PrintStream has print methods for text, but it extends FilterOutputStream, an OutputStream. It is a byte stream that encodes text into bytes.
- **c** ✅ BufferedWriter extends Writer. It buffers characters for another Writer, so it is a character stream.
- **d** ✅ FileReader extends InputStreamReader, which extends Reader, so it reads characters decoded from the file's bytes.
- **e** ❌ FileInputStream extends InputStream and reads raw bytes from a file, with no character decoding.

**The concept.** java.io splits into byte streams (InputStream/OutputStream) and character streams (Reader/Writer).

**Why this is correct.** The quickest test is the superclass: anything that extends Reader or Writer is a character stream, and anything that extends InputStream or OutputStream is a byte stream. FileReader is a Reader and BufferedWriter is a Writer. PrintStream is the trap: it prints text, but it is an OutputStream subclass, which is why System.out is a byte stream that encodes characters for you. DataInputStream and FileInputStream are InputStreams.

**How to reason it out**

1. Recall the four abstract roots: InputStream, OutputStream, Reader, Writer.
2. Classes named ...Reader or ...Writer descend from Reader or Writer.
3. Classes named ...Stream descend from InputStream or OutputStream, even PrintStream.
4. Pick the two Reader/Writer subclasses.

> **Exam tip:** Reader/Writer = characters; InputStream/OutputStream = bytes. PrintStream prints text but is a byte stream.

</details>

📖 Learn this topic: [Java I/O Streams: Console and File Readers, Writers and Streams (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-io-streams-console-file-readers-writers/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 19

*Performing Input and Output Operations Using the Java I/O API*

**Given:
```java
import java.io.*;

class Point {
    int x = 3, y = 4;
}

public class Main {
    public static void main(String[] args) throws Exception {
        var bytes = new ByteArrayOutputStream();
        try (var out = new ObjectOutputStream(bytes)) {
            System.out.print("open ");
            out.writeObject(new Point());
            System.out.print("written");
        }
    }
}
```
What is the result?** Choose one.

- **a.** Compilation fails at the writeObject call
- **b.** open written is printed
- **c.** open is printed, then a NotSerializableException is thrown
- **d.** open is printed, then an InvalidClassException is thrown

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ writeObject takes any Object, so the compiler accepts a Point. The Serializable check happens at run time.
- **b** ❌ Point does not implement Serializable, so writeObject cannot complete and "written" is never printed.
- **c** ✅ The stream opens and prints "open"; writeObject then finds that Point is not Serializable and throws java.io.NotSerializableException: Point.
- **d** ❌ InvalidClassException reports a problem with a class during deserialization, such as a serialVersionUID mismatch. Writing a non-serializable object throws NotSerializableException.

**The concept.** Only objects whose class implements java.io.Serializable can be written with ObjectOutputStream, and the check happens at run time.

**Why this is correct.** ObjectOutputStream.writeObject(Object) accepts any reference, so the program compiles. When it runs, the stream checks the object's class; Point does not implement Serializable, so writeObject throws NotSerializableException naming the class. "open" was already printed, so the output is "open" followed by the exception. InvalidClassException is the deserialization-side error for an unusable class, not the write-side one.

**How to reason it out**

1. Confirm the code compiles: writeObject takes an Object parameter.
2. Check whether Point implements Serializable: it does not.
3. writeObject throws NotSerializableException at run time.
4. Anything printed before the call still appears.

> **Exam tip:** Serializable is checked when writeObject runs, not by the compiler: a non-serializable object throws NotSerializableException.

</details>

📖 Learn this topic: [Java Serialization and Deserialization: Serializable, transient and serialVersionUID (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-serialization-deserialization-serializable-transient/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

## Question 20

*Developing Applications with Localization Support*

**Given:
```java
import java.util.Locale;

public class Main {
    public static void main(String[] args) {
        Locale loc = Locale.of("fr", "CA");
        System.out.print(loc + " ");
        System.out.print(loc.toLanguageTag());
    }
}
```
What is the result?** Choose one.

- **a.** fr_CA fr_CA
- **b.** fr-CA fr-CA
- **c.** fr-CA fr_CA
- **d.** fr_CA fr-CA

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ toLanguageTag() never uses an underscore: BCP 47 language tags separate their subtags with hyphens.
- **b** ❌ toString() is not a language tag. It uses the legacy language_COUNTRY form with an underscore.
- **c** ❌ The two forms are reversed. toString() joins language and country with an underscore; toLanguageTag() produces the BCP 47 tag with a hyphen.
- **d** ✅ toString() prints the language and country joined by an underscore, and toLanguageTag() prints the IETF BCP 47 tag, which uses a hyphen.

**The concept.** A Locale has two textual forms: toString() (language_COUNTRY) and toLanguageTag() (an IETF BCP 47 tag such as fr-CA).

**Why this is correct.** Locale.of("fr", "CA") creates French as used in Canada. Its toString() is fr_CA, the form used in resource bundle file names. toLanguageTag() converts it to the BCP 47 tag fr-CA, the form accepted by Locale.forLanguageTag. The program prints fr_CA fr-CA.

**How to reason it out**

1. Locale.of(language, country) builds the locale; the arguments are normalized to lower-case language and upper-case country.
2. String concatenation calls toString(), which joins the parts with an underscore.
3. toLanguageTag() returns the BCP 47 form, which joins the parts with a hyphen.

> **Exam tip:** toString() gives fr_CA (bundle file names); toLanguageTag() gives fr-CA (BCP 47).

</details>

📖 Learn this topic: [Java Localization: Locales, Resource Bundles and Formatting Numbers, Dates and Messages (1Z0-831)](https://www.savemycert.com/revision/oracle-java-se-developer/java-localization-resource-bundles-number-date-formatting/?utm_source=github&utm_medium=readme&utm_campaign=1z0-831-study-guide)

---

[← Back to the 1Z0-831 study guide](README.md)
