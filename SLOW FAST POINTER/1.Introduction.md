
# 🐢🐇 Slow-Fast Pointer Pattern

> A complete guide to the **Slow-Fast Pointer / Two-Pointer technique** for LeetCode problems.

---

## 📑 Table of Contents

1. [What is Slow-Fast Pointer?](#1-what-is-slow-fast-pointer)
2. [Why do we use it?](#2-why-do-we-use-it)
3. [The mental model](#3-the-mental-model)
4. [Main types](#4-main-types)
5. [Type 1: Speed Difference](#type-1-speed-difference)
6. [Type 2: Fixed Gap](#type-2-fixed-gap)
7. [Type 3: Read / Write](#type-3-read--write-pointer)
8. [Type 4: State Jumping](#type-4-state-jumping)
9. [Type 5: Combination](#type-5-slow-fast--other-techniques)
10. [How to recognize problems](#how-to-recognize-slow-fast-problems)
11. [Common mistakes](#common-mistakes)
12. [Complexity](#complexity)
13. [Interview explanation](#interview-explanation)
14. [Practice order](#recommended-leetcode-practice-order)
15. [Final cheat sheet](#final-cheat-sheet)

---

## 1. What is Slow-Fast Pointer?

**Slow-Fast Pointer** is a two-pointer technique. Two pointers move through a data structure and keep a **useful relationship** between them.

The most common version:

```text
slow -> moves 1 step
fast -> moves 2 steps
```

But it is **not limited to 1-step and 2-step movement**. The pointers can also keep:

- Different speeds
- A fixed gap
- Different jobs (read / write)
- State transitions (`state -> next state`)

### Core Idea

> **Use two pointers to keep a relationship that gives useful information without traversing the data again and again.**

---

## 2. Why do we use it?

Take this linked list:

```text
1 -> 2 -> 3 -> 4 -> 5
```

To find the middle with a simple approach:

```text
1. Count the length
2. Walk again to length / 2
```

That needs **two passes**.

With Slow-Fast, we need only **one pass**:

```text
slow -> +1
fast -> +2
```

When `fast` reaches the end, `slow` is at the middle.

Most Slow-Fast solutions give:

```text
Time  : O(n)
Space : O(1)
```

> Note: for "find middle", brute force is also O(n) time and O(1) space. Slow-Fast just does it in one pass. The real big win is in problems like cycle detection, where the alternative is a HashSet with O(n) space.

---

## 3. The mental model

Do NOT memorize:

> "Slow-Fast means slow moves 1 and fast moves 2."

That is only one form. Remember this instead:

> **Two pointers keep a relationship that helps us solve the problem efficiently.**

The relationship can be:

```text
Different Speed         slow +1, fast +2
Fixed Distance          fast is k positions ahead of slow
Read / Write            fast reads, slow writes
State Jumping           slow = next(state), fast = next(next(state))
```

---

## 4. Main types

| Type | Slow Pointer | Fast Pointer | Common Use |
|---|---|---|---|
| 1. Speed Difference | +1 | +2 | Middle, Cycle, Cycle Entrance |
| 2. Fixed Gap | Normal | k positions ahead | Kth from end, Remove Nth from end |
| 3. Read / Write | Writes | Reads | Remove duplicates, Remove element |
| 4. State Jumping | 1 state | 2 states | Happy Number, any state cycle |
| 5. Combination | Finds middle | Supports it | Palindrome, Reorder, Sort List |

---

#  TYPE 1: Speed Difference

Basic movement:

```cpp
slow = slow->next;
fast = fast->next->next;
```

---

## 5. Find Middle of Linked List

**LC 876: Middle of the Linked List**

```text
1 -> 2 -> 3 -> 4 -> 5
          ^
         slow
```

When `fast` reaches the end, `slow` is at the middle.

### Template

```cpp
ListNode* slow = head;
ListNode* fast = head;

while(fast != NULL && fast->next != NULL){
    slow = slow->next;
    fast = fast->next->next;
}

return slow;
```

### Odd vs Even length

```text
Odd  : 1 -> 2 -> 3 -> 4 -> 5    -> returns 3 (the only middle)
Even : 1 -> 2 -> 3 -> 4         -> returns 3 (the SECOND middle)
```

If you need the **first middle** for even length (needed in Reorder List and Sort List), change the loop condition:

```cpp
while(fast->next != NULL && fast->next->next != NULL){
    slow = slow->next;
    fast = fast->next->next;
}
```

### Complexity

```text
Time  : O(n)
Space : O(1)
```

---

## 6. Detect Linked List Cycle

**LC 141: Linked List Cycle**

```text
1 -> 2 -> 3 -> 4 -> 5
          ^         |
          |         v
          8 <- 7 <- 6
```

- No cycle: `fast` reaches `NULL`
- Cycle: `slow` and `fast` meet

### Template

```cpp
ListNode* slow = head;
ListNode* fast = head;

while(fast != NULL && fast->next != NULL){
    slow = slow->next;
    fast = fast->next->next;

    if(slow == fast){
        return true;
    }
}

return false;
```

### Why do they meet?

Think of a circular track.

- Once both are inside the cycle, `fast` gains **exactly 1 step** on `slow` every iteration.
- So the gap shrinks by 1 each time: `... 3, 2, 1, 0`.
- Fast can never jump over slow, so the gap must hit 0. That is the meeting.

### Important Rule

```cpp
if(slow == fast)      // compares NODE ADDRESSES (correct)
```

not:

```cpp
if(slow->val == fast->val)   // compares values (WRONG)
```

### Complexity

```text
Time  : O(n)
Space : O(1)
```

---

## 7. Find Beginning of Cycle

**LC 142: Linked List Cycle II**

Two phases:

```text
Phase 1 -> Find the meeting point
Phase 2 -> Find the cycle entrance
```

### Phase 1: Find meeting point

```cpp
while(fast && fast->next){
    slow = slow->next;
    fast = fast->next->next;

    if(slow == fast)
        break;
}
```

If there is no cycle, return `NULL`.

### Phase 2: Find entrance

Reset `slow` to `head`. Now move **both** one step at a time. Where they meet is the entrance.

### Complete Template

```cpp
ListNode* slow = head;
ListNode* fast = head;

// Phase 1
while(fast && fast->next){
    slow = slow->next;
    fast = fast->next->next;

    if(slow == fast)
        break;
}

// No cycle
if(fast == NULL || fast->next == NULL)
    return NULL;

// Phase 2
slow = head;

while(slow != fast){
    slow = slow->next;
    fast = fast->next;
}

return slow;
```

### Why does Phase 2 work? (Proof)

Let:

```text
a = distance from head to cycle entrance
b = distance from entrance to meeting point
c = distance from meeting point back to entrance
(cycle length = b + c)
```

At the meeting point:

```text
slow travelled = a + b
fast travelled = 2(a + b)
```

Fast travelled extra full loops, so:

```text
2(a + b) = (a + b) + k(b + c)
a + b    = k(b + c)
a        = k(b + c) - b
a        = (k - 1)(b + c) + c
```

So walking `a` steps from `head` is the same as walking `c` steps from the meeting point (plus some full loops). Both land on the **entrance**.

### Remember

```text
Phase 1: slow +1, fast +2   -> meeting point
Phase 2: slow = head, both +1 -> cycle entrance
```

---

#  TYPE 2: Fixed Gap

The pointers do not need different speeds. Instead:

> **Fast starts a fixed distance ahead of slow.**

Useful for:

```text
kth node from the end
nth node from the end
```

---

## 8. Kth Node From End

```text
1 -> 2 -> 3 -> 4 -> 5
```

2nd node from the end is `4`.

**Step 1:** Move `fast` k steps ahead.

**Step 2:** Move both together until `fast` is `NULL`.

### Template

```cpp
ListNode* slow = head;
ListNode* fast = head;

// Create a gap of k
for(int i = 0; i < k; i++){
    if(fast == NULL) return NULL;   // k is bigger than list length
    fast = fast->next;
}

// Move together
while(fast != NULL){
    slow = slow->next;
    fast = fast->next;
}

return slow;
```

### Core Idea

```text
First : fast = k positions ahead
Then  : slow +1, fast +1
```

The gap stays `k` the whole time.

---

## 9. Remove Nth Node From End

**LC 19: Remove Nth Node From End of List**

Same fixed-gap idea. A **dummy node** handles deleting the head cleanly.

### Why `i <= n` (n + 1 steps)?

We want `slow` to stop **one node before** the target, so we can do `slow->next = slow->next->next`. That needs a gap of `n + 1`.

### Template

```cpp
ListNode* dummy = new ListNode(0);
dummy->next = head;

ListNode* slow = dummy;
ListNode* fast = dummy;

// Create gap of n + 1
for(int i = 0; i <= n; i++){
    fast = fast->next;
}

// Move together
while(fast != NULL){
    slow = slow->next;
    fast = fast->next;
}

// Delete target
ListNode* target = slow->next;
slow->next = target->next;
delete target;

return dummy->next;
```

---

#  TYPE 3: Read / Write Pointer

Common in arrays.

```text
fast -> reads / scans
slow -> writes valid elements
```

They do not move at fixed speeds.

---

## 10. Remove Duplicates From Sorted Array

**LC 26: Remove Duplicates from Sorted Array**

```text
Input : 1 1 2 2 3
Valid : 1 2 3
```

### Mental Model

```text
fast -> scanner
slow -> last written unique element
```

### Template

```cpp
if(nums.empty()) return 0;

int slow = 0;

for(int fast = 1; fast < nums.size(); fast++){
    if(nums[fast] != nums[slow]){
        slow++;
        nums[slow] = nums[fast];
    }
}

return slow + 1;
```

> Here `slow` points to the **last valid element**, so the count is `slow + 1`.
> In LC 27 below, `slow` points to the **next empty position**, so the count is `slow`. Do not mix these up.

---

## 11. Remove Element

**LC 27: Remove Element**

```text
nums = [3, 2, 2, 3], val = 3
Valid part: 2 2
```

### Template

```cpp
int slow = 0;

for(int fast = 0; fast < nums.size(); fast++){
    if(nums[fast] != val){
        nums[slow] = nums[fast];
        slow++;
    }
}

return slow;
```

### General Read/Write Template

```cpp
int slow = 0;

for(int fast = 0; fast < n; fast++){
    if(valid(nums[fast])){
        nums[slow] = nums[fast];
        slow++;
    }
}

return slow;
```

---

## 12. Why does Read/Write work?

```text
fast -> scans everything
slow -> marks the end of the valid part
```

> Fast finds useful data. Slow decides where to store it.

---

#  TYPE 4: State Jumping

Instead of a linked list:

```text
node -> next node
```

we can have:

```text
state -> next state
```

If every state has **exactly one** next state, we can use cycle detection.

---

## 13. Happy Number

**LC 202: Happy Number**

```text
19 -> 82 -> 68 -> 100 -> 1
```

```text
1^2 + 9^2 = 82
8^2 + 2^2 = 68
6^2 + 8^2 = 100
1^2 + 0^2 + 0^2 = 1
```

Some numbers fall into a cycle and never reach 1. So the chain of numbers works like a linked list.

### Define next()

```cpp
int nextNumber(int n){
    int sum = 0;

    while(n > 0){
        int digit = n % 10;
        sum += digit * digit;
        n /= 10;
    }

    return sum;
}
```

### Template

```cpp
int slow = n;
int fast = n;

do{
    slow = nextNumber(slow);
    fast = nextNumber(nextNumber(fast));
} while(slow != fast);

return slow == 1;
```

> Why `slow == 1` at the end? `1 -> 1` is a self-loop, so a happy number also ends in a cycle (of length 1).

---

## 14. General State-Cycle Template

When the problem looks like:

```text
state -> next state -> next state -> ...
```

Ask: **"Can this eventually repeat?"** If yes, try:

```cpp
slow = start;
fast = start;

while(true){
    slow = next(slow);
    fast = next(next(fast));

    if(slow == fast)
        break;
}
```

> This loop is safe only when a cycle is guaranteed (finite states, every state has a next). Otherwise add an end check.

---

#  TYPE 5: Slow-Fast + Other Techniques

Sometimes Slow-Fast is only **one step** of a bigger solution.

---

## 15. Palindrome Linked List

**LC 234: Palindrome Linked List**

```text
1 -> 2 -> 2 -> 1
```

Combine 3 techniques:

```text
1. Slow-Fast        -> find middle
2. Reverse          -> reverse second half
3. Compare          -> check both halves
```

### Full Template

```cpp
// Step 1: Find middle
ListNode* slow = head;
ListNode* fast = head;

while(fast && fast->next){
    slow = slow->next;
    fast = fast->next->next;
}

// Step 2: Reverse second half
ListNode* prev = NULL;

while(slow){
    ListNode* next = slow->next;
    slow->next = prev;
    prev = slow;
    slow = next;
}

// Step 3: Compare
ListNode* left = head;
ListNode* right = prev;

while(right){
    if(left->val != right->val)
        return false;

    left = left->next;
    right = right->next;
}

return true;
```

> This changes the original list. If the problem needs the list unchanged, reverse the second half back after comparing.

---

## 16. Reorder List

**LC 143: Reorder List**

```text
Input  : 1 -> 2 -> 3 -> 4 -> 5
Output : 1 -> 5 -> 2 -> 4 -> 3
```

Combines:

```text
Slow-Fast (first middle) + Reverse + Merge
```

### Template

```cpp
void reorderList(ListNode* head){
    if(!head || !head->next) return;

    // Step 1: Find FIRST middle
    ListNode* slow = head;
    ListNode* fast = head;

    while(fast->next && fast->next->next){
        slow = slow->next;
        fast = fast->next->next;
    }

    // Step 2: Split
    ListNode* second = slow->next;
    slow->next = NULL;

    // Step 3: Reverse second half
    ListNode* prev = NULL;

    while(second){
        ListNode* next = second->next;
        second->next = prev;
        prev = second;
        second = next;
    }

    // Step 4: Merge alternately
    ListNode* first = head;
    second = prev;

    while(second){
        ListNode* t1 = first->next;
        ListNode* t2 = second->next;

        first->next = second;
        second->next = t1;

        first = t1;
        second = t2;
    }
}
```

---

## 17. Sort List

**LC 148: Sort List**

Use Slow-Fast to find the middle, split, then merge sort.

```cpp
ListNode* getMid(ListNode* head){
    ListNode* slow = head;
    ListNode* fast = head->next;     // starts one ahead -> first middle

    while(fast && fast->next){
        slow = slow->next;
        fast = fast->next->next;
    }

    return slow;
}

// Inside sortList:
// ListNode* mid = getMid(head);
// ListNode* right = mid->next;
// mid->next = NULL;      // split
// then sort(head), sort(right), and merge
```

> Slow-Fast is often a **supporting technique** inside a bigger algorithm.
> Top-down merge sort uses O(log n) recursion stack, so this solution is O(n log n) time and O(log n) space.

---

#  How to Recognize Slow-Fast Problems

Ask these questions while reading a problem.

| # | Question | Think | Examples |
|---|---|---|---|
| 1 | Do I need the middle? | slow +1, fast +2 | LC 876, 234, 143, 148 |
| 2 | Can the structure have a cycle? | slow +1, fast +2 | LC 141, 142 |
| 3 | Do I need kth/nth from the end? | fast gets k-step head start | LC 19 |
| 4 | Do I need to remove/keep array elements in place? | fast scans, slow writes | LC 26, 27 |
| 5 | Does every state lead to exactly one next state? | try cycle detection | LC 202 |

---

## 📊 Pattern Recognition Chart

| Problem Clue | Pattern | Pointer Relationship |
|---|---|---|
| Find middle | Speed Difference | +1 / +2 |
| Detect cycle | Speed Difference | +1 / +2 |
| Cycle entrance | Two Phase | +1 / +2, then reset slow |
| Kth from end | Fixed Gap | Fast k ahead |
| Remove nth from end | Fixed Gap | Fast n + 1 ahead (with dummy) |
| Remove duplicates | Read/Write | Fast reads, slow writes |
| Remove element | Read/Write | Fast reads, slow writes |
| Happy Number | State Cycle | next / next(next) |
| Palindrome | Combination | Middle + Reverse + Compare |
| Reorder List | Combination | Middle + Reverse + Merge |
| Sort List | Combination | Middle + Merge Sort |

---

##  Master Decision Tree

```text
                    Need two pointers?
                            |
          +-----------------+-----------------+
          |                 |                 |
     Linked list?      Array in-place?    Repeating states?
          |                 |                 |
          v                 v                 v
    What do I need?    Read / Write       State cycle
          |            (fast reads,       (next / next(next))
          |             slow writes)
   +------+-------+---------+
   |              |         |
 Middle        Cycle?    Kth from end
   |              |         |
 +1 / +2      +1 / +2    Fixed gap
              (+ reset
               for entrance)
```

---

## ⚙️ Common Templates

### Template A: Middle

```cpp
ListNode* slow = head;
ListNode* fast = head;

while(fast && fast->next){
    slow = slow->next;
    fast = fast->next->next;
}

return slow;
```

### Template B: Cycle Detection

```cpp
ListNode* slow = head;
ListNode* fast = head;

while(fast && fast->next){
    slow = slow->next;
    fast = fast->next->next;

    if(slow == fast)
        return true;
}

return false;
```

### Template C: Cycle Entrance

```cpp
ListNode* slow = head;
ListNode* fast = head;

while(fast && fast->next){
    slow = slow->next;
    fast = fast->next->next;

    if(slow == fast)
        break;
}

if(!fast || !fast->next)
    return NULL;

slow = head;

while(slow != fast){
    slow = slow->next;
    fast = fast->next;
}

return slow;
```

### Template D: Kth From End

```cpp
ListNode* slow = head;
ListNode* fast = head;

for(int i = 0; i < k; i++){
    if(!fast) return NULL;
    fast = fast->next;
}

while(fast){
    slow = slow->next;
    fast = fast->next;
}

return slow;
```

### Template E: Read/Write Array

```cpp
int slow = 0;

for(int fast = 0; fast < n; fast++){
    if(valid(nums[fast])){
        nums[slow] = nums[fast];
        slow++;
    }
}

return slow;
```

### Template F: State Cycle

```cpp
slow = start;
fast = start;

while(true){
    slow = next(slow);
    fast = next(next(fast));

    if(slow == fast)
        break;
}
```

---

# ⚠️ Common Mistakes

### Mistake 1: Forgetting `fast->next`

Wrong:

```cpp
while(fast != NULL){
    fast = fast->next->next;   // crashes if fast->next is NULL
}
```

Correct:

```cpp
while(fast && fast->next)
```

The order matters. Check `fast` first, then `fast->next`.

### Mistake 2: Dereferencing without checking

Always check a pointer before using `->`, for example `fast->next`, `slow->next->next`, or `head->next` on an empty list.

### Mistake 3: Confusing the middle position

For an even-length list `1 -> 2 -> 3 -> 4`, the middles are `2` and `3`.

```text
fast = head,       condition fast && fast->next            -> second middle (3)
fast = head,       condition fast->next && fast->next->next -> first middle (2)
```

Know which middle the problem needs.

### Mistake 4: Comparing values instead of nodes

```cpp
if(slow == fast)               // correct: same node
if(slow->val == fast->val)     // wrong: same value, different node
```

### Mistake 5: Thinking fast always moves twice

Not always. Slow-Fast can also mean:

```text
+1 / +2            speed difference
k-distance gap     fixed gap
read / write       array compression
next / next(next)  state jumping
```

### Mistake 6: Wrong gap in Remove Nth from End

Gap of `n` stops `slow` **on** the target. Gap of `n + 1` stops it **before** the target. For deletion you need `n + 1` (with a dummy node).

---

#  Complexity

Most Slow-Fast solutions:

```text
Time  : O(n)
Space : O(1)
```

But always check the **complete** algorithm:

| Problem | Time | Space |
|---|---|---|
| Middle / Cycle / Cycle entrance | O(n) | O(1) |
| Kth / Remove Nth from end | O(n) | O(1) |
| Remove duplicates / element | O(n) | O(1) |
| Palindrome (reverse in place) | O(n) | O(1) |
| Reorder List | O(n) | O(1) |
| Sort List (top-down merge sort) | O(n log n) | O(log n) stack |

---

#  Brute Force vs Slow-Fast

| Problem | Brute Force | Slow-Fast |
|---|---|---|
| Middle | Count, then walk again: O(n) time, O(1) space, 2 passes | O(n) time, O(1) space, 1 pass |
| Cycle detect | HashSet of visited nodes: O(n) time, **O(n) space** | O(n) time, **O(1) space** |
| Kth from end | Count length, then walk: 2 passes | 1 pass |

---

#  Interview Explanation

**"What is Slow-Fast Pointer?"**

> "Slow-Fast Pointer is a two-pointer technique where we keep a useful relationship between two pointers, often moving one slower than the other. It is commonly used in linked lists to find the middle, detect cycles, find the cycle entrance, or keep a fixed gap such as finding the kth node from the end. It usually solves these problems in O(n) time and O(1) extra space."

---

#  One-Line Memory Tricks

| Problem | Trick |
|---|---|
| Middle | Slow +1, Fast +2 |
| Cycle | Slow +1, Fast +2, check meeting |
| Cycle Entrance | Meet, reset slow to head, move both +1 |
| Kth From End | Fast gets k-step head start |
| Remove Nth From End | Dummy node, gap of n + 1 |
| Array Remove/Keep | Fast reads, slow writes |
| Happy Number | Treat numbers as linked-list states |
| Palindrome | Middle, Reverse, Compare |
| Reorder List | Middle, Reverse, Merge |

---

# 🧪 Recommended LeetCode Practice Order

### 🟢 Level 1: Understand the Pattern

1. **LC 876**: Middle of the Linked List
2. **LC 141**: Linked List Cycle
3. **LC 26**: Remove Duplicates from Sorted Array
4. **LC 27**: Remove Element

### 🟡 Level 2: Modify the Pattern

5. **LC 19**: Remove Nth Node From End of List
6. **LC 202**: Happy Number
7. **LC 142**: Linked List Cycle II

### 🟠 Level 3: Combine Techniques

8. **LC 234**: Palindrome Linked List
9. **LC 143**: Reorder List
10. **LC 148**: Sort List

---

#  Final Cheat Sheet

```text
                    SLOW-FAST POINTER
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
    Speed Difference   Fixed Gap       Read / Write
          |                |                |
          v                v                v
       +1 / +2         Fast k ahead     Fast reads
          |                |            Slow writes
          v                v                v
    +-----------+       Kth end        Array cleanup
    |           |
    v           v
  Middle      Cycle
                |
                v
          Cycle Entrance
```

Extension:

```text
state -> next(state) -> next(state) -> ... -> possible cycle
slow = +1 state
fast = +2 states
```

---

#  The Real Skill

Don't ask:

> "Which Slow-Fast code should I memorize?"

Ask:

> **"What relationship can I keep between two pointers so that I get the answer without repeated work?"**

### The 5 relationships to remember

```text
1. Speed
   slow +1, fast +2

2. Distance
   fast = slow + k

3. Position
   slow = write position
   fast = scanner

4. State
   slow = next(state)
   fast = next(next(state))

5. Combination
   Slow-Fast + Reverse + Merge/Compare
```

These five cover a large part of the **Slow-Fast / Two-Pointer problems** you will see on LeetCode.
