# The right way to reverse a String in Java

## Facts and Terminology

As you probably know, Java uses UTF-16 to represent `String`s. The `char` data type and the `Character` class are based on the original Unicode specification, which defined characters as fixed-width 16-bit entities. The Unicode Standard has since been changed to allow for characters whose representation requires more than 16 bits.
Therefore, in the UTF-16 representation, there are characters (Code Points) which are represented by one- and some other characters which are represented by two char values (Code Units).

Please check out the [Java String length confusion](Java_String_length/Java_String_length.md) article and the [JavaDoc of the Character class](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html) for more details and a better explanation.

## Example

Character: `A`  
"UTF-16 representation" in Java: `"\u0041"`

Character: `𝔸` ([Mathematical double-struck capital A](https://codepoints.net/U+1D538))  
"UTF-16 representation" in Java: `"\uD835\uDD38"`

The first one is straightforward, the second one is a little bit more interesting; this single character (Code Point) is represented by two [Unicode escapes](https://docs.oracle.com/javase/specs/jls/se11/html/jls-3.html#jls-3.3). This means a couple of things:

- This single character is represented by **two** `char` (or `Character`) values (Code Units)
- The `length()` of this `String` is **two** (see: [Java String length confusion](Java_String_length/Java_String_length.md))
- The `toCharArray()` method returns a char array (`char[]`) which has two elements (`0xD835` and `0xDD38` respectively)
- Both `charAt(0)` and `charAt(1)` return something (no `StringIndexOutOfBoundsException`) but these values are not valid characters
- If you do any character manipulation, you need to consider this case and handle these characters which consist of two `char`s (surrogates)
- Therefore, most of the character manipulation code we ever wrote is probably broken :)

This basically means that you probably do not want to do any character manipulation (see below).

## Broken String reverse

By this point, you might have a good guess what is wrong with this (very commonly used) solution to reverse a String:

```
static String reverse(String original) {
    String reversed = "";
    for (int i = original.length() - 1;  0 <= i; i--) {
        reversed += original.charAt(i);
    }

    return reversed;
}
```

Let's see it in action:

```
String str = "\uD835\uDD38BC"; // Three characters: 𝔸, B, C (4 chars)
System.out.println(str); // prints 𝔸BC
System.out.println(reverse(str)); // prints CB??
```

If you run the reverse method above, it will produce a `String` like this: `"CB\uDD38\uD835"`. `C` and `B` are ok but `\uDD38\uD835` is invalid, that's why you see `??` when you print it. The method should not have reversed them, the valid result would be `"CB\uD835\uDD38"` (`CB𝔸`).

## Solution

Usually, not writing code to solve problems is a good idea:

```
static String reverse(String original) {
    return new StringBuilder(original).reverse().toString();
}
```

If you want to take a (small) step further, here's a Java 8+ one-liner:

```
Function<String, String> reverse = s -> new StringBuilder(s).reverse().toString();
```

If you are curious how is this implemented under-the-hood, check out what StringBuilder's [reverse()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html#reverse()) method does (it is in the `AbstractStringBuilder` class).

So what broken `String` manipulations have you seen?
