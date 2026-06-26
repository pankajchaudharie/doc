# Java Coding Questions — Most Asked in Companies

The programs companies (TCS, Infosys, Wipro, Capgemini, Accenture, Cognizant + product
firms) actually ask in Java coding rounds. Since you're **new to backend**, each problem
has the **idea in plain English**, **runnable code**, the **complexity**, and **what's
being tested**. Practice writing these without an IDE — a lot of interviews still use a
whiteboard or a plain editor.

> Coding-round reality: they care that you (1) write compiling code, (2) handle edge
> cases, (3) know the time complexity, and (4) can explain your logic. Talk while you code.

---

## 0. New-to-backend primer for coding rounds

A few Java basics that trip up frontend devs:
- **Every program needs a class + `main`:**
  ```java
  public class Main {
      public static void main(String[] args) {
          System.out.println("Hello");
      }
  }
  ```
- **Types are mandatory:** `int i = 5;`, `String s = "x";`, `int[] arr = {1, 2, 3};`.
- **String compare:** use `.equals()` for content, **not** `==` (which compares
  references). This is the #1 Java beginner bug.
- **Print:** `System.out.println(...)`. **Length:** arrays use `arr.length` (a field),
  strings use `str.length()` (a method), lists use `list.size()`. Easy to mix up.
- **Useful imports:** `import java.util.*;` gives you `List`, `Map`, `Set`, `Arrays`,
  `Collections`, etc.

---

## Part A — Strings

### Q1. Reverse a string
```java
String reverse(String str) {
    return new StringBuilder(str).reverse().toString();
}
// Without built-ins:
String reverseManual(String str) {
    char[] ch = str.toCharArray();
    int i = 0, j = ch.length - 1;
    while (i < j) { char t = ch[i]; ch[i++] = ch[j--]; t = ch[j+1]; ch[j+1] = t; }
    return new String(ch);
}
```
**Idea:** `StringBuilder.reverse()` is the one-liner; the manual version uses two
pointers swapping inward. **Complexity:** O(n). **Note:** `String` is immutable, so you
build a new one.

### Q2. Check palindrome
```java
boolean isPalindrome(String s) {
    s = s.toLowerCase().replaceAll("[^a-z0-9]", "");
    int i = 0, j = s.length() - 1;
    while (i < j) if (s.charAt(i++) != s.charAt(j--)) return false;
    return true;
}
```
**Idea:** two pointers from both ends. **Complexity:** O(n) time, O(1) extra space.

### Q3. Check anagram
```java
boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] count = new int[26];                 // for lowercase a–z
    for (int i = 0; i < a.length(); i++) {
        count[a.charAt(i) - 'a']++;
        count[b.charAt(i) - 'a']--;
    }
    for (int c : count) if (c != 0) return false;
    return true;
}
```
**Idea:** a 26-slot frequency array — `+1` for one string, `−1` for the other; all
zeros = anagram. **Why asked:** tests the **char-to-index trick** (`ch - 'a'`).

### Q4. Count occurrences of a character
```java
int countChar(String str, char target) {
    int count = 0;
    for (char c : str.toCharArray()) if (c == target) count++;
    return count;
}
```

### Q5. First non-repeating character
```java
Character firstUnique(String s) {
    Map<Character, Integer> count = new LinkedHashMap<>();   // keeps insertion order
    for (char c : s.toCharArray()) count.merge(c, 1, Integer::sum);
    for (var e : count.entrySet()) if (e.getValue() == 1) return e.getKey();
    return null;
}
firstUnique("swiss"); // 'w'
```
**Note:** `LinkedHashMap` preserves order so "first" is meaningful. `merge(c, 1, Integer::sum)`
is the clean way to increment a count.

### Q6. Count vowels and consonants
```java
void countVowelsConsonants(String s) {
    int vowels = 0, consonants = 0;
    for (char c : s.toLowerCase().toCharArray()) {
        if (Character.isLetter(c)) {
            if ("aeiou".indexOf(c) >= 0) vowels++; else consonants++;
        }
    }
    System.out.println(vowels + " vowels, " + consonants + " consonants");
}
```

### Q7. Check if two strings are rotations of each other
```java
boolean isRotation(String a, String b) {
    return a.length() == b.length() && (a + a).contains(b);
}
isRotation("abcd", "cdab"); // true
```
**Idea:** any rotation of `a` is a substring of `a + a`. A neat trick interviewers love.

### Q8. Remove duplicate characters preserving order
```java
String removeDuplicates(String s) {
    StringBuilder sb = new StringBuilder();
    Set<Character> seen = new HashSet<>();
    for (char c : s.toCharArray()) if (seen.add(c)) sb.append(c);
    return sb.toString();
}
removeDuplicates("programming"); // "progamin"
```
**Note:** `seen.add(c)` returns `false` if already present — clean way to dedupe.

### Q9. Find duplicate characters and their counts
```java
void printDuplicates(String s) {
    Map<Character, Integer> count = new HashMap<>();
    for (char c : s.toCharArray()) count.merge(c, 1, Integer::sum);
    count.forEach((k, v) -> { if (v > 1) System.out.println(k + " = " + v); });
}
```

---

## Part B — Arrays

### Q10. Find largest and smallest
```java
void minMax(int[] arr) {
    int min = arr[0], max = arr[0];
    for (int x : arr) { min = Math.min(min, x); max = Math.max(max, x); }
    System.out.println("min=" + min + ", max=" + max);
}
```

### Q11. Second largest in one pass
```java
int secondLargest(int[] arr) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int x : arr) {
        if (x > first) { second = first; first = x; }
        else if (x > second && x != first) second = x;
    }
    return second;
}
```
**Why asked:** "do it without sorting" tests whether you can track two values in a
single scan — O(n) vs the O(n log n) sort.

### Q12. Reverse an array in place
```java
void reverse(int[] arr) {
    int i = 0, j = arr.length - 1;
    while (i < j) { int t = arr[i]; arr[i++] = arr[j]; arr[j--] = t; }
}
```

### Q13. Two Sum
```java
int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();   // value -> index
    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];
        if (seen.containsKey(need)) return new int[]{ seen.get(need), i };
        seen.put(nums[i], i);
    }
    return new int[]{};
}
```
**Idea:** hash map → O(n). **Why asked:** the most common array question; brute force is
two loops (O(n²)), the good answer uses a map.

### Q14. Remove duplicates from an array
```java
int[] unique(int[] arr) {
    return Arrays.stream(arr).distinct().toArray();   // Java 8 streams
}
```

### Q15. Find the missing number in 1..n
```java
int missingNumber(int[] arr, int n) {
    int expected = n * (n + 1) / 2;        // sum of 1..n
    int actual = Arrays.stream(arr).sum();
    return expected - actual;
}
```
**Idea:** sum formula trick → O(n) time, O(1) space, no sorting. A favourite.

### Q16. Find duplicates in an array
```java
void findDuplicates(int[] arr) {
    Set<Integer> seen = new HashSet<>();
    for (int x : arr) if (!seen.add(x)) System.out.println("Duplicate: " + x);
}
```

### Q17. Move all zeros to the end
```java
void moveZeros(int[] arr) {
    int insert = 0;
    for (int x : arr) if (x != 0) arr[insert++] = x;
    while (insert < arr.length) arr[insert++] = 0;
}
```

### Q18. Max subarray sum (Kadane's algorithm)
```java
int maxSubArray(int[] nums) {
    int best = nums[0], current = nums[0];
    for (int i = 1; i < nums.length; i++) {
        current = Math.max(nums[i], current + nums[i]);
        best = Math.max(best, current);
    }
    return best;
}
```
**Why asked:** beats the O(n²) brute force; a well-known interview classic.

### Q19. Rotate an array by k positions
```java
void rotate(int[] arr, int k) {
    k %= arr.length;
    reverse(arr, 0, arr.length - 1);
    reverse(arr, 0, k - 1);
    reverse(arr, k, arr.length - 1);
}
void reverse(int[] a, int i, int j) {
    while (i < j) { int t = a[i]; a[i++] = a[j]; a[j--] = t; }
}
```
**Idea:** the "reverse three times" trick — O(n) time, O(1) space.

---

## Part C — Numbers

### Q20. Check prime
```java
boolean isPrime(int n) {
    if (n < 2) return false;
    for (int i = 2; i <= Math.sqrt(n); i++) if (n % i == 0) return false;
    return true;
}
```
**Key point:** loop only to `√n`. State this — it's the optimization being tested.

### Q21. Factorial
```java
long factorial(int n) {
    long result = 1;
    for (int i = 2; i <= n; i++) result *= i;
    return result;
}
```
**Watch-out:** use `long` — `int` overflows past 12! (mention this maturity point).

### Q22. Fibonacci series
```java
void fibonacci(int n) {
    long a = 0, b = 1;
    for (int i = 0; i < n; i++) { System.out.print(a + " "); long t = a + b; a = b; b = t; }
}
```

### Q23. Reverse a number / check palindrome number
```java
int reverseNumber(int n) {
    int rev = 0;
    while (n != 0) { rev = rev * 10 + n % 10; n /= 10; }
    return rev;
}
boolean isPalindromeNumber(int n) { return n == reverseNumber(n); }
```
**Idea:** `% 10` peels the last digit, `/ 10` drops it — the core digit-manipulation
pattern.

### Q24. Sum of digits
```java
int sumOfDigits(int n) {
    int sum = 0;
    n = Math.abs(n);
    while (n != 0) { sum += n % 10; n /= 10; }
    return sum;
}
```

### Q25. Armstrong number (e.g., 153 = 1³+5³+3³)
```java
boolean isArmstrong(int n) {
    int original = n, sum = 0, digits = String.valueOf(n).length();
    while (n != 0) { int d = n % 10; sum += Math.pow(d, digits); n /= 10; }
    return sum == original;
}
```

### Q26. Swap two numbers without a third variable
```java
a = a + b;  b = a - b;  a = a - b;
```

### Q27. FizzBuzz
```java
for (int i = 1; i <= 100; i++) {
    String out = "";
    if (i % 3 == 0) out += "Fizz";
    if (i % 5 == 0) out += "Buzz";
    System.out.println(out.isEmpty() ? i : out);
}
```

---

## Part D — Collections (very common for backend roles)

### Q28. Count word frequency in a sentence
```java
Map<String, Integer> wordCount(String sentence) {
    Map<String, Integer> map = new HashMap<>();
    for (String word : sentence.toLowerCase().split("\\s+"))
        map.merge(word, 1, Integer::sum);
    return map;
}
```
**Real-world:** log analysis, search indexing — a practical backend task.

### Q29. Sort a Map by value
```java
Map<String, Integer> sortByValue(Map<String, Integer> map) {
    return map.entrySet().stream()
        .sorted(Map.Entry.comparingByValue(Comparator.reverseOrder()))
        .collect(Collectors.toMap(
            Map.Entry::getKey, Map.Entry::getValue,
            (a, b) -> a, LinkedHashMap::new));   // LinkedHashMap keeps sorted order
}
```
**Why asked:** combines streams + comparators + map collectors — a strong Java 8 signal.

### Q30. Sort a list of objects by a field (Comparator)
```java
record Employee(String name, int salary) {}

List<Employee> sortBySalary(List<Employee> list) {
    return list.stream()
        .sorted(Comparator.comparingInt(Employee::salary).reversed())
        .collect(Collectors.toList());
}
```
**Real-world:** sorting any domain list (movies by rating, orders by date). Be ready to
sort by multiple fields: `.comparing(A).thenComparing(B)`.

### Q31. Remove duplicates from a List
```java
List<Integer> unique = list.stream().distinct().collect(Collectors.toList());
// or simply: new ArrayList<>(new LinkedHashSet<>(list));
```

### Q32. Find the highest-paid employee per department (group + reduce)
```java
Map<String, Optional<Employee>> topPerDept(List<Employee> emps) {
    return emps.stream().collect(Collectors.groupingBy(
        e -> e.dept(),
        Collectors.maxBy(Comparator.comparingInt(Employee::salary))));
}
```
**Why asked:** `groupingBy` + a downstream collector is the most-asked Java 8 streams
pattern for experienced roles.

### Q33. Convert a List to a Map
```java
Map<String, Integer> map = employees.stream()
    .collect(Collectors.toMap(Employee::name, Employee::salary));
```

---

## Part E — Java 8 Streams quick wins (interviewers love these)

```java
// Sum of even numbers
int sum = list.stream().filter(n -> n % 2 == 0).mapToInt(Integer::intValue).sum();

// Count strings longer than 3 chars
long count = words.stream().filter(w -> w.length() > 3).count();

// Comma-joined string
String csv = list.stream().map(String::valueOf).collect(Collectors.joining(", "));

// Max value
int max = list.stream().max(Integer::compare).orElseThrow();

// Find first match
Optional<String> first = words.stream().filter(w -> w.startsWith("a")).findFirst();
```
**Talk track:** streams = declarative pipelines (like JS `map/filter/reduce`). Mention
they're **lazy** and only run when a terminal operation (`collect`, `sum`, `count`) is hit.

---

## Part F — Output / concept questions

```java
String a = "hello";
String b = "hello";
System.out.println(a == b);        // true  — same string pool reference
System.out.println(a.equals(b));   // true  — same content

String c = new String("hello");
System.out.println(a == c);        // false — new object, different reference
System.out.println(a.equals(c));   // true  — same content
```
**Explain:** `==` compares references, `.equals()` compares content. String literals
share the **string pool**; `new String()` forces a new heap object. The single most
common Java gotcha.

```java
System.out.println('a' + 'b');     // 195  — chars promoted to int (97 + 98)
System.out.println("" + 'a' + 'b'); // "ab" — string concatenation
System.out.println(10 / 3);        // 3    — integer division
System.out.println(10.0 / 3);      // 3.333... — double division
```

---

## Part G — More problems (round 2)

### Q34. GCD and LCM of two numbers
```java
int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }   // Euclid's algorithm
int lcm(int a, int b) { return (a / gcd(a, b)) * b; }
```
**Idea:** Euclid's algorithm — `gcd(a,b) = gcd(b, a % b)`. **Why asked:** classic recursion
+ a math optimization (`/gcd` before `*b` avoids overflow).

### Q35. Find the longest substring without repeating characters
```java
int longestUnique(String s) {
    Map<Character, Integer> last = new HashMap<>();
    int start = 0, best = 0;
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (last.containsKey(c) && last.get(c) >= start) start = last.get(c) + 1;
        last.put(c, i);
        best = Math.max(best, i - start + 1);
    }
    return best;
}
```
**Idea:** sliding window — O(n). A very common medium-level question.

### Q36. Merge two sorted arrays
```java
int[] merge(int[] a, int[] b) {
    int[] out = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) out[k++] = a[i] <= b[j] ? a[i++] : b[j++];
    while (i < a.length) out[k++] = a[i++];
    while (j < b.length) out[k++] = b[j++];
    return out;
}
```
**Idea:** two pointers — O(n + m). The merge step of merge sort.

### Q37. Bubble sort (be ready to write a sort by hand)
```java
void bubbleSort(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++)
        for (int j = 0; j < arr.length - 1 - i; j++)
            if (arr[j] > arr[j + 1]) { int t = arr[j]; arr[j] = arr[j + 1]; arr[j + 1] = t; }
}
```
**Complexity:** O(n²) — mention that real code uses `Arrays.sort()` (O(n log n)); this just
proves you know a sorting algorithm.

### Q38. Binary search (on a sorted array)
```java
int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;            // avoids integer overflow
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) lo = mid + 1; else hi = mid - 1;
    }
    return -1;
}
```
**Complexity:** O(log n). **Why asked:** tests the off-by-one boundaries and the overflow-
safe mid calculation.

### Q39. Count occurrences of each word and find the most frequent
```java
String mostFrequentWord(String text) {
    Map<String, Integer> count = new HashMap<>();
    for (String w : text.toLowerCase().split("\\s+")) count.merge(w, 1, Integer::sum);
    return count.entrySet().stream()
        .max(Map.Entry.comparingByValue())
        .map(Map.Entry::getKey).orElse(null);
}
```
**Why asked:** real backend task (log/search analytics) + streams `max`.

### Q40. Check if a number is a power of two
```java
boolean isPowerOfTwo(int n) { return n > 0 && (n & (n - 1)) == 0; }
```
**Idea:** a power of two has exactly one set bit; `n & (n-1)` clears it to 0. A neat
bit-manipulation trick interviewers like.

### Q41. Count set bits in an integer
```java
int countBits(int n) {
    int count = 0;
    while (n != 0) { n &= (n - 1); count++; }   // clears the lowest set bit each loop
    return count;
}
// Built-in: Integer.bitCount(n)
```

### Q42. Reverse a singly linked list
```java
class Node { int val; Node next; Node(int v) { val = v; } }

Node reverse(Node head) {
    Node prev = null;
    while (head != null) {
        Node next = head.next;   // save
        head.next = prev;        // reverse pointer
        prev = head;             // advance prev
        head = next;             // advance head
    }
    return prev;
}
```
**Why asked:** the most common linked-list question — pointer juggling in O(n), O(1) space.

### Q43. Find the factorial using BigInteger (large numbers)
```java
import java.math.BigInteger;
BigInteger factorial(int n) {
    BigInteger result = BigInteger.ONE;
    for (int i = 2; i <= n; i++) result = result.multiply(BigInteger.valueOf(i));
    return result;
}
```
**Talk track:** shows maturity — `long` overflows past 20!, `BigInteger` doesn't.

### Q44. Remove a given character from a string
```java
String removeChar(String s, char c) {
    return s.chars()
        .filter(ch -> ch != c)
        .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
        .toString();
}
// Simpler: s.replace(String.valueOf(c), "")
```

### Q45. Find common elements in two arrays (intersection)
```java
List<Integer> intersection(int[] a, int[] b) {
    Set<Integer> set = Arrays.stream(a).boxed().collect(Collectors.toSet());
    return Arrays.stream(b).filter(set::contains).distinct().boxed().collect(Collectors.toList());
}
```
**Idea:** a `Set` gives O(1) lookups → O(n) instead of nested loops.

---

### Interview tips for Java coding rounds
1. **Compile in your head** — semicolons, types, `length` vs `length()` vs `size()`.
2. **Use `.equals()` for objects/strings**, `==` only for primitives.
3. **Mention complexity** and the optimization (`√n`, hash map, single pass).
4. **Prefer Java 8 streams** for collection transforms — shows modern Java.
5. **Handle edge cases**: empty array, null, single element, overflow (`int` → `long`).
6. **Say your approach first**, then code — interviewers grade reasoning.
