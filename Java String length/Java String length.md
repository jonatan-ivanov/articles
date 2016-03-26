#Java String length
***Confusion about supplementary characters***

#Facts and Terminology
As you probably know, Java uses *UTF-16* to represent ***String***s. In order to understand the confusion about ***String.length()***, you need to be familiar with some Encoding/Unicode terms.

**Code Point**: A unique integer value which represents a character in the code space.

**Code Unit**: A bit sequence used to encode characters (Code Points). One or more Code Units may be required to represent a Code Point.

#UTF-16
Unicode Code Points are logically divided into 17 planes. The first plane, the Basic Multilingual Plane (BMP) contains the "classic" characters (from ```U+0000``` to ```U+FFFF```). The other planes contain the supplementary characters (from ```U+10000``` to ```U+10FFFF```).

**Characters (Code Points) from the first plane are encoded in one 16-bit Code Unit with the same value. Supplementary characters (Code Points) are encoded in two Code Units** (encoding-specific, see [Wiki](http://en.wikipedia.org/wiki/UTF-16 "UTF-16") for the explanation).

##Example
Character: ```A```  
Unicode Code Point: ```U+0041```  
UTF-16 Code Unit(s): ```0041```

Character: [```Mathematical double-struck capital A```](http://codepoints.net/U+1D538)  
Unicode Code Point: ```U+1D538```  
UTF-16 Code Unit(s): ```D835 DD38```

As you can see here, there are characters which are encoded in two Code Units.

#String.length()
Let's take a look at the Javadoc of the [```length()```](http://docs.oracle.com/javase/8/docs/api/java/lang/String.html#length--) method:

>```public int length()```  
Returns the length of this string. The length is equal to the number of [Unicode code units](http://docs.oracle.com/javase/8/docs/api/java/lang/Character.html#unicode) in the string.

So if you have **one** supplementary character which consists of **two** code units, the length of that single character is **two**.

```java
//Mathematical double-struck capital A
String str = "\uD835\uDD38";
System.out.println(str);
System.out.println(str.length()); //prints 2
```

Which is correct according to the documentation, but maybe it's not expected.

#~Solution
You need to count the code points, not the code units:

```
String str = "\uD835\uDD38";
System.out.println(str);
System.out.println(str.codePointCount(0, str.length()));
```

See: [```codePointCount(int beginIndex, int endIndex)```](http://docs.oracle.com/javase/8/docs/api/java/lang/String.html#codePointCount-int-int-)

#References/Sources
- [The Java Language Specification](http://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.1)
- [Unicode Glossary: Code Point](http://unicode.org/glossary/#code_point)
- [Wiki: Code Point](http://en.wikipedia.org/wiki/Code_point)
- [Unicode Glossary: Code Unit](http://unicode.org/glossary/#code_unit)
- [Wiki: Code Unit](http://en.wikipedia.org/wiki/Code_unit#Code_unit)
- [Wiki: Unicode](http://en.wikipedia.org/wiki/Unicode)
- [Wiki: UTF-16](http://en.wikipedia.org/wiki/Utf-16)
- [Supplementary Characters in the Java Platform](http://www.oracle.com/technetwork/articles/javase/supplementary-142654.html)
- [Wiki: Unicode Planes](http://en.wikipedia.org/wiki/Plane_%28Unicode%29)
