# Technical Refresher Program for SMTS/LMTS Interviews

## 8-Week Intensive Study Plan - UPDATED Part 1 (Weeks 1-4)

**Last Updated:** December 2024 - With NeetCode & System Design Primer Integration

**Goal:** Master technical fundamentals for Senior/Lead Member of Technical Staff interviews

**Time Commitment:** 15-20 hours per week

- Weekdays: 2 hours/day (M-F = 10 hours)
- Weekends: 5-10 hours total

---

## 🎯 Part 1 Resources Overview

### For Algorithms (Weeks 1-2):

**Primary:** NeetCode 150

- Website: https://neetcode.io/practice
- Roadmap: https://neetcode.io/roadmap
- YouTube: https://www.youtube.com/@NeetCode
- **Why:** Pattern-based, video explanations, curated problems

**Supplementary:**

- VisualGo: https://visualgo.net/ (visualize algorithms)
- LeetCode Patterns: https://seanprashad.com/leetcode-patterns/

### For System Design (Weeks 3-4):

**Primary Theory:** System Design Primer

- GitHub: https://github.com/donnemartin/system-design-primer
- **Why:** Comprehensive, free, includes Anki flashcards

**Primary Videos:**

- Gaurav Sen: https://www.youtube.com/@gkcs
- ByteByteGo: https://www.youtube.com/@ByteByteGo

**Practice:**

- Excalidraw: https://excalidraw.com/ (drawing tool)

**Case Studies:**

- Netflix: https://netflixtechblog.com/
- Uber: https://www.uber.com/blog/engineering/
- Salesforce: https://engineering.salesforce.com/

---

# PHASE 1: ALGORITHMS & DATA STRUCTURES (Weeks 1-2)

## Week 1: Core Data Structures

### 📚 Learning Objectives

By end of week, you should be able to:

- Implement all core data structures from scratch in Java
- Analyze time/space complexity accurately
- Solve 10+ LeetCode medium problems
- Follow NeetCode roadmap pattern-by-pattern

### 🎯 Week 1 Strategy

- **Primary Resource:** NeetCode 150 list
- **Watch videos:** Only AFTER attempting each problem
- **Track progress:** On https://neetcode.io/practice
- **Create flashcards:** For each pattern in Anki

---

### Monday - Arrays & Hash Tables (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** Review Arrays & Hashing section (10 min)
    - Link: https://neetcode.io/roadmap
    - Understand the pattern structure
- [ ] **Watch:** NeetCode "Arrays & Hashing" intro (20 min)
    - Link: https://www.youtube.com/watch?v=KLlXCFG5TnA
    - Don't code along yet - just understand concepts
- [ ] **Read:** System Design Primer - Hash Tables (15 min)
    - Link: https://github.com/donnemartin/system-design-primer#hash-table
    - Understand use cases at scale
- [ ] **Review:** Java HashMap internals (15 min)
    - Link: https://www.baeldung.com/java-hashmap

**Evening Practice (1 hour)**

**All problems from NeetCode 150:**

- [ ] **Problem 1: Contains Duplicate** (LeetCode #217) - 10 min
    
    - Link: https://neetcode.io/problems/duplicate-integer
    - Attempt first, then watch video if stuck
- [ ] **Problem 2: Two Sum** (LeetCode #1) - 15 min
    
    - Link: https://neetcode.io/problems/two-sum
    - Classic HashMap problem
    - Watch: https://www.youtube.com/watch?v=KLlXCFG5TnA (if stuck)
- [ ] **Problem 3: Group Anagrams** (LeetCode #49) - 20 min
    
    - Link: https://neetcode.io/problems/anagram-groups
    - Practice: HashMap with sorted string key
- [ ] **Problem 4: Top K Frequent Elements** (LeetCode #347) - 15 min
    
    - Link: https://neetcode.io/problems/top-k-elements-in-list
    - Preview of heap pattern (Week 1 Thursday)

**After solving, watch NeetCode solutions for any you struggled with**

**Create 3 Flashcards in Anki:**

1. Q: When to use HashMap vs TreeMap? A: HashMap for O(1) lookup, TreeMap for sorted order
2. Q: Hash collision resolution methods? A: Chaining (LinkedList) vs Open Addressing (probing)
3. Q: HashMap time complexity? A: Average O(1), Worst O(n) when all keys hash to same bucket

---

### Tuesday - LinkedLists & Stacks/Queues (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** Review Linked List section (5 min)
    - Link: https://neetcode.io/roadmap
- [ ] **Watch:** NeetCode "Linked List" intro (20 min)
    - Link: https://www.youtube.com/@NeetCode (search "Linked List")
- [ ] **Visualize:** LinkedList operations on VisualGo (20 min)
    - Link: https://visualgo.net/en/list
    - Try: Insert, delete, reverse operations
    - See the pointer changes visually
- [ ] **Watch:** Princeton Algorithms - Stacks/Queues (15 min)
    - Link: https://www.youtube.com/watch?v=TIC1gappbP8
    - Skip if short on time

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Reverse Linked List** (LeetCode #206) - 15 min
    
    - Link: https://neetcode.io/problems/reverse-linked-list
    - Iterative and recursive solutions
    - Watch NeetCode video after attempting
- [ ] **Problem 2: Merge Two Sorted Lists** (LeetCode #21) - 20 min
    
    - Link: https://neetcode.io/problems/merge-two-sorted-linked-lists
    - Practice pointer manipulation
- [ ] **Problem 3: Linked List Cycle** (LeetCode #141) - 10 min
    
    - Link: https://neetcode.io/problems/linked-list-cycle
    - Fast & slow pointer technique
- [ ] **Problem 4: Valid Parentheses** (LeetCode #20) - 15 min
    
    - Link: https://neetcode.io/problems/validate-parentheses
    - Stack application

**Review solutions on NeetCode for any stuck problems**

**Create 3 Flashcards:**

1. Q: LinkedList vs ArrayList? A: LinkedList O(1) insert/delete, O(n) access. ArrayList opposite.
2. Q: When to use Stack vs Queue? A: Stack for LIFO (undo, parsing), Queue for FIFO (BFS, scheduling)
3. Q: Fast & slow pointer use? A: Detect cycles, find middle element

---

### Wednesday - Trees & Binary Search Trees (2 hours)

**Morning Study (1 hour)**

- [x] **NeetCode Roadmap:** Review Trees section (5 min)
- [x] **Watch:** NeetCode "Trees" introduction (25 min)
    - Link: https://www.youtube.com/@NeetCode (search "Trees")
    - Focus on: DFS, BFS, tree properties
- [x] **Visualize:** Tree traversals on VisualGo (20 min)
    - Link: https://visualgo.net/en/bst
    - Try all traversals: In-order, Pre-order, Post-order, Level-order
    - Understand when each is useful
- [x] **Read:** Balanced trees overview (10 min)
    - AVL, Red-Black basics (don't need deep knowledge)

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Invert Binary Tree** (LeetCode #226) - 10 min
    
    - Link: https://neetcode.io/problems/invert-a-binary-tree
    - Simple recursion problem
- [ ] **Problem 2: Maximum Depth** (LeetCode #104) - 10 min
    
    - Link: https://neetcode.io/problems/depth-of-binary-tree
    - Practice DFS
- [ ] **Problem 3: Diameter of Binary Tree** (LeetCode #543) - 15 min
    
    - Link: https://neetcode.io/problems/binary-tree-diameter
    - Slightly more complex
- [ ] **Problem 4: Validate BST** (LeetCode #98) - 25 min
    
    - Link: https://neetcode.io/problems/valid-binary-search-tree
    - Important BST property check

**Watch NeetCode solutions for deeper understanding**

**Create 4 Flashcards:**

1. Q: DFS vs BFS for trees? A: DFS for path problems, BFS for level-order/shortest-path
2. Q: BST property? A: Left subtree < node < Right subtree (for all nodes)
3. Q: In-order traversal of BST gives? A: Sorted array
4. Q: Balanced tree time complexity? A: O(log n) for search/insert/delete

---

### Thursday - Heaps & Priority Queues (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** Review Heap/Priority Queue section (5 min)
- [ ] **Watch:** NeetCode "Heaps" explanation (20 min)
    - Link: https://www.youtube.com/@NeetCode (search "Heap")
- [ ] **Visualize:** Heap operations on VisualGo (20 min)
    - Link: https://visualgo.net/en/heap
    - Try: Insert, extract-min, heapify
    - See how heap maintains structure
- [ ] **Read:** Java PriorityQueue (15 min)
    - Link: https://www.baeldung.com/java-priority-queue

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Kth Largest in Stream** (LeetCode #703) - 20 min
    
    - Link: https://neetcode.io/problems/kth-largest-integer-in-a-stream
    - Min-heap application
- [ ] **Problem 2: Last Stone Weight** (LeetCode #1046) - 15 min
    
    - Link: https://neetcode.io/problems/last-stone-weight
    - Max-heap simulation
- [ ] **Problem 3: Meeting Rooms II** (LeetCode #253) - 25 min ⭐ **CRITICAL FOR FSL**
    
    - Link: https://neetcode.io/problems/meeting-schedule-ii
    - **THIS IS APPOINTMENT SCHEDULING!**
    - Master this pattern - directly applies to your work
    - Watch NeetCode solution carefully

**This week's most important problem for your domain!**

**Create 3 Flashcards:**

1. Q: Heap operations complexity? A: Insert O(log n), Extract-min O(log n), Peek O(1)
2. Q: Min-heap vs Max-heap? A: Min-heap for smallest element, Max-heap for largest
3. Q: Why heap for "top K"? A: Maintain K-size heap, better than sorting entire array

---

### Friday - Graphs Fundamentals (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** Review Graphs section (5 min)
- [ ] **Watch:** NeetCode "Graphs" introduction (25 min)
    - Link: https://www.youtube.com/@NeetCode (search "Graphs")
    - Focus on: BFS, DFS, when to use each
- [ ] **Visualize:** Graph traversals on VisualGo (20 min)
    - Link: https://visualgo.net/en/dfsbfs
    - Try both BFS and DFS
    - See the search order difference
- [ ] **Read:** Graph representations (10 min)
    - Adjacency List vs Adjacency Matrix

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Number of Islands** (LeetCode #200) - 20 min
    
    - Link: https://neetcode.io/problems/count-number-of-islands
    - DFS/BFS on 2D grid
- [ ] **Problem 2: Clone Graph** (LeetCode #133) - 20 min
    
    - Link: https://neetcode.io/problems/clone-graph
    - HashMap + DFS
- [ ] **Problem 3: Course Schedule** (LeetCode #207) - 20 min ⭐ **DEPENDENCY MANAGEMENT**
    
    - Link: https://neetcode.io/problems/course-schedule
    - Topological sort / cycle detection
    - **Applies to your MP-to-Core migration!**

**Watch NeetCode videos - graph problems are tricky at first**

**Create 3 Flashcards:**

1. Q: Graph representation trade-offs? A: Adj List O(V+E) space, good for sparse. Adj Matrix O(V²), good for dense.
2. Q: When DFS vs BFS? A: DFS for paths/cycles, BFS for shortest path
3. Q: Graph traversal complexity? A: O(V + E) for both DFS and BFS

---

### 🎯 Saturday Deep Dive (4 hours)

**Morning (2 hours): Applied Data Structures**

**Exercise 1: Appointment Conflict Detector (60 min)**

```java
/**
 * Apply Meeting Rooms II pattern from Thursday!
 * This is DIRECTLY applicable to FSL scheduling
 */
Problem: Given List<Appointment>, find all conflicts
- Appointment has: startTime, endTime, resourceId
- Return: List<Pair<Appointment, Appointment>> conflicts

Your approach:
1. Code initial solution (25 min)
   - Think: How is this like Meeting Rooms II?
   - Can you adapt that algorithm?

2. Watch NeetCode "Meeting Rooms II" again (10 min)
   - Link: https://neetcode.io/problems/meeting-schedule-ii
   - Focus on the min-heap approach

3. Optimize your solution (15 min)
   - Target: O(n log n) using sorting + heap
   - Handle edge cases: same start/end times

4. Write test cases (10 min)
```

**Exercise 2: Design Resource Priority Queue (60 min)**

```java
/**
 * Apply Heap pattern for resource allocation
 */
Problem: Assign incoming appointments to best-available resource
- Resources have: skills, location, current_load
- Appointments need: skills, location, priority

Your task:
1. Design the data structure (20 min)
   - Which heap? Min or Max?
   - What's the comparison key?
   - How to handle multiple criteria?

2. Implement in Java (30 min)
   - Use PriorityQueue with custom Comparator
   - Write assignAppointment() method
   - Write releaseResource() method

3. Test with sample data (10 min)
   - Create 5 resources, 10 appointments
   - Verify correct assignment
```

**Afternoon (2 hours): Review & Flashcards**

- [ ] **Review all 13 problems solved this week** (60 min)
    
    - Open each on NeetCode
    - Can you solve them again without looking?
    - Re-watch any videos for problems you struggled with
- [ ] **Watch any skipped NeetCode videos** (30 min)
    
    - Even if you solved the problem, watch the video
    - Learn the optimal approach
    - Note any tricks you missed
- [ ] **Create/Review Anki flashcards** (30 min)
    
    - Install Anki: https://apps.ankiweb.net/
    - Create a deck called "Data Structures"
    - Add all your flashcards from this week
    - Review them (spaced repetition starts now!)

---

### 🎯 Sunday Review & Checkpoint (3 hours)

**Morning (90 min): Connect to Your Projects**

**Write detailed answers (500 words each):**

**1. ESO-Anonymizer Data Structures Analysis**

- What data structures did you actually use?
- Why HashMap for PII field mappings vs TreeMap?
- How did you handle relationship preservation?
    - Is this a graph problem? (objects reference each other)
    - Did you use DFS/BFS to traverse relationships?
- If processing 100M records instead of 10M, what would you change?
    - Different data structures?
    - Batching strategy?

**2. MP-to-Core Migration**

- How would you detect circular dependencies in object relationships?
    - This is Course Schedule problem! (Friday)
    - Topological sort application?
- What's the correct processing order for migrating objects?
    - Graph traversal order?
- Caching strategy for frequently accessed mappings?
    - LRU cache? (Week 1 Monday pattern)

**Afternoon (90 min): Week 1 Checkpoint Test**

**⏱️ Timed Challenge (60 min strict):**

Solve these 3 problems WITHOUT looking at solutions:

- [ ] **Merge K Sorted Lists** (LeetCode #23) - Hard
    
    - Link: https://neetcode.io/problems/merge-k-sorted-linked-lists
    - Combines: LinkedList + Heap
    - Time: 25 minutes
- [ ] **LRU Cache** (LeetCode #146) - Medium ⭐ **CACHING**
    
    - Link: https://neetcode.io/problems/lru-cache
    - Combines: HashMap + LinkedList
    - Time: 20 minutes
- [ ] **Binary Tree Level Order Traversal** (LeetCode #102) - Medium
    
    - Link: https://neetcode.io/problems/binary-tree-level-order-traversal
    - Uses: BFS with queue
    - Time: 15 minutes

**⚠️ Rules:**

- No looking at solutions or hints
- Write production-quality code
- Think out loud (record yourself if possible)
- Explain your approach before coding

**Self-Review (30 min):**

Answer honestly:

- [ ] Could you solve all 3 in 60 minutes?
- [ ] Did you write clean, readable code?
- [ ] Did you explain your approach before coding?
- [ ] Did you handle edge cases?
- [ ] Did you analyze time/space complexity?

**If NO to any:**

- Identify which pattern you're weak on
- Solve 3 more problems from that pattern
- Watch the corresponding NeetCode videos again

---

### 📊 Week 1 Checkpoint

**You should be able to:**

- ✅ Implement any basic data structure from scratch in Java
- ✅ Solve NeetCode medium problems in 20-25 minutes
- ✅ Explain time/space complexity for every operation
- ✅ Choose correct data structure for any problem
- ✅ Write production-quality code with edge case handling

**Problems Solved:** 13+ from NeetCode 150 **Flashcards Created:** 16 cards in Anki **Study Time:** ~16 hours **NeetCode Progress:** Track on https://neetcode.io/practice

**🎯 Week 1 Success Metrics:**

- Can recognize pattern within 2-3 minutes
- Can code solution in 20 minutes
- Can explain why your solution is optimal

---

## Week 2: Algorithm Patterns & Problem-Solving

### 📚 Learning Objectives

By end of week, you should be able to:

- Recognize and apply 10+ algorithm patterns
- Solve scheduling/optimization problems
- Write algorithms with optimal time/space complexity
- Handle 3 medium problems in 60 minutes consistently

### 🎯 Week 2 Strategy

- Continue with **NeetCode 150** list
- Focus on **pattern recognition** over memorization
- Apply patterns to **FSL scheduling problems**
- Start timing yourself (interview simulation)

---

### Monday - Two Pointers & Sliding Window (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** Two Pointers section (5 min)
    - Link: https://neetcode.io/roadmap
- [ ] **Watch:** NeetCode "Two Pointers" pattern (20 min)
    - Link: https://www.youtube.com/@NeetCode (search "Two Pointers")
- [ ] **Watch:** NeetCode "Sliding Window" pattern (20 min)
    - Link: https://www.youtube.com/@NeetCode (search "Sliding Window")
- [ ] **Read:** Pattern recognition guide (15 min)
    - Link: https://leetcode.com/discuss/study-guide/1688903/
    - When to recognize these patterns

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Valid Palindrome** (LeetCode #125) - 10 min
    
    - Link: https://neetcode.io/problems/is-palindrome
    - Basic two pointers
- [ ] **Problem 2: Container With Most Water** (LeetCode #11) - 15 min
    
    - Link: https://neetcode.io/problems/max-water-container
    - Greedy + two pointers
- [ ] **Problem 3: Longest Substring Without Repeating** (LeetCode #3) - 20 min
    
    - Link: https://neetcode.io/problems/longest-substring-without-duplicates
    - Sliding window + HashMap
- [ ] **Problem 4: Minimum Window Substring** (LeetCode #76) - 15 min
    
    - Link: https://neetcode.io/problems/minimum-window-with-characters
    - Advanced sliding window

**Create 3 Flashcards:**

1. Q: Two pointers pattern trigger? A: Sorted array, palindrome check, pair sum
2. Q: Sliding window fixed vs variable? A: Fixed for "subarray of size K", Variable for "smallest/longest subarray"
3. Q: Sliding window time complexity? A: O(n) - each element visited at most twice

---

### Tuesday - Binary Search Variations (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** Binary Search section (5 min)
- [ ] **Watch:** NeetCode "Binary Search" introduction (25 min)
    - Link: https://www.youtube.com/@NeetCode (search "Binary Search")
    - Focus on: Search space reduction
- [ ] **Read:** Binary search on answer space (20 min)
    - Link: https://leetcode.com/discuss/study-guide/786126/
- [ ] **Practice:** Write binary search template from memory (10 min)

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Binary Search** (LeetCode #704) - 10 min
    
    - Link: https://neetcode.io/problems/binary-search
    - Master the template first
- [ ] **Problem 2: Search in Rotated Sorted Array** (LeetCode #33) - 20 min
    
    - Link: https://neetcode.io/problems/search-in-rotated-sorted-array
    - Modified binary search
- [ ] **Problem 3: Find Minimum in Rotated Array** (LeetCode #153) - 15 min
    
    - Link: https://neetcode.io/problems/find-minimum-in-rotated-sorted-array
- [ ] **Problem 4: Time Based Key-Value Store** (LeetCode #981) - 15 min
    
    - Link: https://neetcode.io/problems/time-based-key-value-store
    - Binary search on timestamps ⭐ **VERSIONING**

**Create 3 Flashcards:**

1. Q: Binary search template? A: while(left <= right) { mid = left + (right-left)/2; ...}
2. Q: When to use binary search? A: Sorted array, or search space can be ordered/reduced
3. Q: Time complexity? A: O(log n) - halve search space each iteration

---

### Wednesday - DFS/BFS Deep Dive (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** Review Graph section again (5 min)
- [ ] **Watch:** NeetCode "Graph DFS/BFS" deep dive (30 min)
    - Link: https://www.youtube.com/@NeetCode (search "DFS BFS")
- [ ] **Read:** Backtracking vs DFS (15 min)
    - Link: https://leetcode.com/explore/learn/card/recursion-ii/
- [ ] **Practice:** Write DFS and BFS templates (10 min)

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Word Search** (LeetCode #79) - 20 min
    
    - Link: https://neetcode.io/problems/search-for-word
    - Backtracking on 2D grid
- [ ] **Problem 2: Pacific Atlantic Water Flow** (LeetCode #417) - 20 min
    
    - Link: https://neetcode.io/problems/pacific-atlantic-water-flow
    - DFS from edges
- [ ] **Problem 3: Course Schedule II** (LeetCode #210) - 20 min
    
    - Link: https://neetcode.io/problems/course-schedule-ii
    - Topological sort ⭐ **DEPENDENCY ORDERING**

**Create 3 Flashcards:**

1. Q: DFS vs BFS space complexity? A: DFS O(h) height, BFS O(w) width
2. Q: When is DFS better? A: Finding paths, detecting cycles, tree problems
3. Q: Backtracking template? A: Make choice → Explore → Undo choice

---

### Thursday - Greedy Algorithms (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** Greedy section (5 min)
- [ ] **Watch:** NeetCode "Greedy Algorithms" (25 min)
- [ ] **Watch:** Greedy Algorithm Principles (20 min)
    - Link: https://www.youtube.com/watch?v=HzeK7g8cD0Y
- [ ] **Read:** How to prove greedy is correct (10 min)

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Jump Game** (LeetCode #55) - 15 min
    
    - Link: https://neetcode.io/problems/jump-game
- [ ] **Problem 2: Meeting Rooms** (LeetCode #252) - 10 min ⭐ **SCHEDULING**
    
    - Link: https://neetcode.io/problems/meeting-schedule
    - Simpler version of problem from Week 1
- [ ] **Problem 3: Non-overlapping Intervals** (LeetCode #435) - 20 min ⭐ **SCHEDULING**
    
    - Link: https://neetcode.io/problems/non-overlapping-intervals
    - Core scheduling algorithm
- [ ] **Problem 4: Minimum Arrows to Burst Balloons** (LeetCode #452) - 15 min
    
    - Link: https://neetcode.io/problems/minimum-number-of-arrows-to-burst-balloons
    - Interval greedy pattern

**⭐ Critical: All interval/scheduling problems use greedy approach!**

**Create 3 Flashcards:**

1. Q: How to prove greedy? A: Show greedy choice property + optimal substructure
2. Q: Greedy vs DP? A: Greedy makes local optimal choice, DP considers all choices
3. Q: Interval scheduling pattern? A: Sort by end time, greedily select non-overlapping

---

### Friday - Dynamic Programming Basics (2 hours)

**Morning Study (1 hour)**

- [ ] **NeetCode Roadmap:** 1-D DP section (5 min)
- [ ] **Watch:** NeetCode "Dynamic Programming" introduction (30 min)
    - Link: https://www.youtube.com/@NeetCode (search "Dynamic Programming")
- [ ] **Read:** Memoization vs Tabulation (15 min)
    - Link: https://www.geeksforgeeks.org/tabulation-vs-memoization/
- [ ] **Practice:** Identify DP pattern (10 min)

**Evening Practice (1 hour)**

**All from NeetCode 150:**

- [ ] **Problem 1: Climbing Stairs** (LeetCode #70) - 10 min
    
    - Link: https://neetcode.io/problems/climbing-stairs
    - Classic DP intro
- [ ] **Problem 2: House Robber** (LeetCode #198) - 15 min
    
    - Link: https://neetcode.io/problems/house-robber
    - State machine DP
- [ ] **Problem 3: Coin Change** (LeetCode #322) - 20 min
    
    - Link: https://neetcode.io/problems/coin-change
    - Unbounded knapsack
- [ ] **Problem 4: Longest Increasing Subsequence** (LeetCode #300) - 15 min
    
    - Link: https://neetcode.io/problems/longest-increasing-subsequence
    - Classic LIS

**Create 4 Flashcards:**

1. Q: How to identify DP? A: Optimal substructure + overlapping subproblems
2. Q: Memoization vs Tabulation? A: Top-down (recursive) vs Bottom-up (iterative)
3. Q: DP time/space? A: Usually O(n²) time, can optimize space to O(n)
4. Q: DP state definition? A: dp[i] = "answer to subproblem ending at i"

---

### 🎯 Saturday Applied Algorithms (5 hours)

**Morning (3 hours): Scheduling Algorithm Design**

**Exercise 1: Design Appointment Scheduler (90 min)**

```
Problem: Design algorithm to schedule N appointments to M resources

Constraints:
- Each appointment requires specific skills
- Resources have availability windows  
- Minimize total travel distance
- Respect priority levels

Part 1 (30 min): Naive O(N*M) solution
- For each appointment, check all resources
- Choose first available that matches skills
- Code this up

Part 2 (30 min): Optimize with data structures
- Use priority queue for resource selection
- Use interval trees for availability check
- What's the new complexity?

Part 3 (30 min): Add constraint handling
- Handle skill matching with bitmasks
- Handle time windows with binary search
- What patterns from Week 2 apply?

Write:
- Complete Java implementation
- Time/space complexity analysis
- Trade-offs discussion
- When would this break? (failure modes)
```

**Exercise 2: Route Optimization Algorithm (90 min)**

```
Problem: Given N service appointments, find optimal visit order

Constraints:
- Each has time window
- Travel time between locations
- Technician has shift hours
- Some have dependencies

Questions to answer:
1. Is this TSP? If so, can we use heuristics?
2. Greedy solution vs optimal solution?
   - Apply Thursday's interval scheduling!
3. How to handle real-time changes?
4. What if 1000 appointments? Does algorithm scale?

Implement:
1. Greedy solution (30 min)
   - Sort by earliest start time
   - Greedily select next closest appointment
   
2. Handle dependencies (30 min)
   - Model as DAG (Directed Acyclic Graph)
   - Topological sort first
   - Then apply greedy on sorted order
   
3. Test and analyze (30 min)
   - Create test cases
   - Compare greedy vs brute force (small N)
   - Measure performance
```

**Afternoon (2 hours): Pattern Recognition Practice**

**Exercise 3: Pattern Matching (60 min)**

For each problem, identify the pattern BEFORE solving:

1. Subarray Sum Equals K → ?
2. Find Peak Element → ?
3. Rotate Array → ?
4. Merge Intervals → ?
5. Word Break → ?
6. Alien Dictionary → ?
7. Find Duplicate Number → ?
8. Shortest Path in Grid → ?
9. Combination Sum → ?
10. Trapping Rain Water → ?

**Answers:**

1. Sliding Window + HashMap/Prefix Sum
2. Binary Search
3. Reversal Algorithm (Three reversals)
4. Sort + Greedy (Interval pattern)
5. Dynamic Programming (Word segmentation)
6. Topological Sort (Graph)
7. Cycle Detection (Floyd's)
8. BFS (Shortest path)
9. Backtracking
10. Two Pointers / Stack

**Practice:**

- Open each problem on NeetCode
- Identify pattern in 2 minutes
- If wrong, watch the video to learn why

**Exercise 4: Create Pattern Cheat Sheet (60 min)**

Build a 1-page reference document:

```
PATTERN RECOGNITION CHEAT SHEET

1. TWO POINTERS
   Triggers: Sorted array, palindrome, pair sum
   Template: left=0, right=n-1, while left<right...
   Problems: Container With Water, Valid Palindrome

2. SLIDING WINDOW
   Triggers: Subarray, substring, "longest/shortest"
   Template: left=0, for right in array, while invalid...
   Problems: Longest Substring, Minimum Window

3. BINARY SEARCH
   Triggers: Sorted array, "find in O(log n)", search space reduction
   Template: while left<=right, mid=left+(right-left)/2...
   Problems: Search Rotated Array, Time Based KV

[Continue for all patterns...]

15. INTERVALS/SCHEDULING
   Triggers: Time ranges, meetings, appointments
   Strategy: Sort by end time, greedy select
   Problems: Meeting Rooms, Non-overlapping Intervals
   **FSL APPLICATION: Appointment scheduling**
```

---

### 🎯 Sunday Integration & Mock (4 hours)

**Morning (2 hours): Connect to FSL**

**Write detailed 500-word answers:**

**1. Real-Time Schedule Optimization**

- How does current FSL scheduling work? (What you know)
- Where would you apply DP vs Greedy?
    - DP for: Resource allocation with future consideration?
    - Greedy for: Immediate appointment assignment?
- How to handle 1000+ appointments/minute?
    - Batch processing?
    - Stream processing?
- What data structures are needed?
    - Priority queues for resource selection?
    - Interval trees for conflict detection?

**2. Resource Assignment Algorithm**

- Skills matching approach
    - Exact match vs partial match?
    - How to score matches?
- Location-based optimization
    - Geohashing for proximity?
    - Distance calculations?
- Priority queue design
    - What's the priority key?
    - Multiple heaps for different priorities?
- Handle capacity constraints
    - How to track resource availability?
    - Update after each assignment?

**3. Dependency Resolution**

- Prerequisites between service appointments
    - Model as DAG
    - Topological sort for ordering
- Circular dependency detection
    - DFS with visited states
    - Error handling
- Error handling strategy
    - What if cycles detected?
    - How to report to user?

**Afternoon (2 hours): Timed Mock Coding Interview**

**⏱️ Mock Interview Setup:**

- Set timer for 45 minutes total
- Use whiteboard or paper (NO IDE autocomplete)
- Explain approach out loud BEFORE coding
- Handle follow-up questions

**Problem Set (choose 2, solve BOTH in 45 min):**

- [ ] **Design Add and Search Words Data Structure** (LeetCode #211)
    
    - Link: https://neetcode.io/problems/design-word-search-data-structure
    - Combines: Trie + DFS
- [ ] **Task Scheduler** (LeetCode #621) ⭐ **RESOURCE ALLOCATION**
    
    - Link: https://neetcode.io/problems/task-scheduling
    - Combines: Heap + Greedy
    - DIRECTLY applicable to FSL!
- [ ] **Alien Dictionary** (LeetCode #269) ⭐ **DEPENDENCY ORDERING**
    
    - Link: https://neetcode.io/problems/foreign-dictionary
    - Combines: Graph + Topological Sort
    - Applies to object migration ordering!

**After Mock (30 min):**

- Review your solutions
- Watch NeetCode videos for both problems
- Identify what you'd do differently
- Write down: "Next time I will..."

**Final Review (30 min):**

- What patterns did you recognize?
- Were you fast enough?
- Did you explain while coding?
- Edge cases handled?

---

### 📊 Week 2 Checkpoint

**You should be able to:**

- ✅ Recognize algorithm patterns in 2-3 minutes
- ✅ Solve medium problems in 20 minutes
- ✅ Explain why your algorithm is optimal
- ✅ Design scheduling algorithms from scratch
- ✅ Connect algorithms to real FSL problems
- ✅ Solve 2 mediums in 45 minutes consistently

**Problems Solved:** 20+ this week (40+ total from NeetCode 150) **Algorithms Mastered:** 8 core patterns **Study Time:** ~19 hours

**Self-Assessment Questions:**

1. Can you solve 2 medium problems in 45 minutes?
    - If NO: Do 5 more problems from weak patterns
2. Can you explain your algorithm before coding?
    - If NO: Practice "think out loud" more
3. Can you optimize brute force to optimal?
    - If NO: Review time/space complexity analysis

**NeetCode Progress:** ~40/150 problems complete

---

# PHASE 2: SYSTEM DESIGN FUNDAMENTALS (Weeks 3-4)

## Week 3: Scalability & Distributed Systems

### 📚 Learning Objectives

By end of week, you should be able to:

- Design systems that scale to millions of users
- Make informed trade-offs (CAP theorem)
- Design effective caching and load balancing
- Complete full system design in 45 minutes
- Use Excalidraw for professional diagrams

### 🎯 Week 3 Strategy

- **Primary Reading:** System Design Primer on GitHub
- **Video Learning:** Gaurav Sen for each topic
- **Practice Tool:** Excalidraw for all designs
- **Real-World Study:** Engineering blogs

---

### Monday - Scalability Fundamentals (2 hours)

**Morning Study (1 hour)**

- [ ] **Primary Reading:** System Design Primer - Scalability (30 min)
    - Link: https://github.com/donnemartin/system-design-primer#scalability
    - Read sections: Performance vs scalability, Latency vs throughput
    - Read: Availability vs consistency (CAP theorem intro)
- [ ] **Video:** Gaurav Sen - "Basics of System Design" (30 min)
    - Link: https://www.youtube.com/watch?v=xpDnVSmNFX0
    - Focus on: Horizontal vs vertical scaling
    - Note: Load balancing introduction

**Alternative video if preferred:**

- [ ] Harvard CS75 - Scalability Lecture
    - Link: https://www.youtube.com/watch?v=-W9F__D3oY4
    - More academic but very thorough

**Evening Practice (1 hour)**

- [ ] **Excalidraw Exercise:** Design URL Shortener (40 min)
    - Open: https://excalidraw.com/
    - Start with 100 users (simple diagram)
    - Evolve to 1M users (add load balancer, caching)
    - Evolve to 100M users (add sharding, CDN)
    - Save your design (bookmark the Excalidraw link!)
- [ ] **Document:** What changes at each scale tier? (20 min)
    - Write in your notes:
    - 100 users: Single server OK
    - 1M users: Add load balancer, DB replicas
    - 100M users: Add caching, CDN, sharding
    - Why each change was necessary

**Create 5 Flashcards (from System Design Primer):**

1. Q: Vertical vs Horizontal scaling? A: Vertical=bigger machine, Horizontal=more machines. Horizontal better for high availability.
2. Q: What breaks first at 10x scale? A: Usually database (add replicas, then shard)
3. Q: Read-heavy vs Write-heavy? A: Read-heavy: add replicas. Write-heavy: shard database.
4. Q: Stateless vs Stateful services? A: Stateless easier to scale (just add servers). Stateful needs sticky sessions or shared state.
5. Q: Master-Slave vs Multi-Master? A: Master-Slave simpler but single point of failure. Multi-Master handles failures but complex conflicts.

---

### Tuesday - Load Balancing & Caching (2 hours)

**Morning Study (1 hour)**

- [ ] **Primary Reading:** System Design Primer - Load Balancer (20 min)
    - Link: https://github.com/donnemartin/system-design-primer#load-balancer
    - Read: Layer 4 vs Layer 7 load balancing
    - Read: Load balancing algorithms
- [ ] **Primary Reading:** System Design Primer - Cache (20 min)
    - Link: https://github.com/donnemartin/system-design-primer#cache
    - Read: Cache-aside, Write-through, Write-behind
    - Read: When to update cache
- [ ] **Video:** ByteByteGo - "Load Balancer Explained" (10 min)
    - Link: https://www.youtube.com/@ByteByteGo (search "Load Balancer")
- [ ] **Video:** Gaurav Sen - "Caching Pitfalls" (10 min)
    - Link: https://www.youtube.com/watch?v=UH7wkvcf0ys

**Evening Practice (1 hour)**

- [ ] **Code Exercise:** Implement LRU Cache with TTL (30 min)
    - Use your Week 1 knowledge!
    - Add time-to-live (TTL) feature
    - Make it thread-safe (bonus)
- [ ] **Excalidraw:** Design load balancer for FSL scheduling API (30 min)
    - Draw the architecture
    - Show: Client → Load Balancer → App Servers → Database
    - Decide: Which LB algorithm? (Round-robin? Least connections? Weighted?)
    - Add: Health checks (how to detect dead servers?)
    - Consider: Sticky sessions needed? (for scheduling context)
    - Save your design

**Create 4 Flashcards:**

1. Q: Cache-aside pattern? A: App checks cache first, on miss loads from DB and updates cache. Good for read-heavy.
2. Q: Write-through vs Write-behind? A: Write-through updates cache+DB together (slower writes, consistency). Write-behind updates cache first, DB later (faster, risk of data loss).
3. Q: Consistent hashing? A: Hash keys and servers to ring. Keys go to next server clockwise. Adding/removing servers only affects neighbors.
4. Q: Load balancer algorithms? A: Round-robin (equal distribution), Least connections (unequal loads), Weighted (different server capacities).

---

### Wednesday - CAP Theorem & Database Scaling (2 hours)

**Morning Study (1 hour)**

- [ ] **Primary Reading:** System Design Primer - CAP Theorem (15 min)
    - Link: https://github.com/donnemartin/system-design-primer#cap-theorem
    - Understand: Consistency, Availability, Partition Tolerance
    - Note: "You can only pick 2"
- [ ] **Video:** Gaurav Sen - "CAP Theorem Simplified" (15 min)
    - Link: https://www.youtube.com/watch?v=k-Yaq8AHlFA
    - Real-world examples
- [ ] **Primary Reading:** System Design Primer - Database (20 min)
    - Link: https://github.com/donnemartin/system-design-primer#database
    - Read: Sharding, Denormalization, SQL tuning
    - Read: NoSQL section
- [ ] **Video:** ByteByteGo - "Database Sharding" (10 min)
    - Visual explanation of sharding strategies

**Evening Practice (1 hour)**

- [ ] **Excalidraw:** Sharding strategy for FSL appointments (40 min)
    - Scenario: 100M appointments need to be sharded
    - Option 1: Shard by Territory (most queries are by territory)
    - Option 2: Shard by Time (appointments are time-based)
    - Option 3: Shard by Resource (appointments belong to resources)
    - Draw all three approaches
    - Analyze: Cross-shard queries? Hot shards? Rebalancing?
    - Choose one and justify
    - Save design
- [ ] **Document:** SQL vs NoSQL for FSL features (20 min)
    - Appointments: SQL or NoSQL? Why?
    - Location tracking: SQL or NoSQL? Why?
    - Audit logs: SQL or NoSQL? Why?
    - Real-time updates: SQL or NoSQL? Why?

**Create 4 Flashcards:**

1. Q: CAP examples? A: CP=Banking (consistency over availability), AP=DNS (availability over consistency), CA=Single node (not distributed).
2. Q: When to shard database? A: When single DB can't handle load (>10M rows, high write throughput, or slow queries despite indexing).
3. Q: Sharding strategies? A: Range (by ID ranges), Hash (consistent hashing), Directory (lookup table for shard mapping).
4. Q: ACID vs BASE? A: ACID=Strong consistency (SQL). BASE=Eventually consistent (NoSQL). Trade consistency for availability/partition tolerance.

---

### Thursday - Distributed Systems Patterns (2 hours)

**Morning Study (1 hour)**

- [ ] **Primary Reading:** System Design Primer - Consistency Patterns (15 min)
    - Link: https://github.com/donnemartin/system-design-primer#consistency-patterns
    - Read: Weak, Eventual, Strong consistency
- [ ] **Primary Reading:** System Design Primer - Availability Patterns (15 min)
    - Link: https://github.com/donnemartin/system-design-primer#availability-patterns
    - Read: Fail-over, Replication
- [ ] **Video:** MIT 6.824 - Distributed Systems Intro (30 min)
    - Link: https://www.youtube.com/watch?v=cQP8WApzIQQ
    - Optional but excellent background
    - Skip if short on time

**Evening Practice (1 hour)**

- [ ] **Excalidraw:** Distributed transaction for appointments (30 min)
    - Scenario: Update resource availability + Create appointment (must be atomic)
    - What if partial failure?
    - Solution 1: Two-phase commit (2PC)
        - Draw: Coordinator → Participants → Commit/Rollback
        - Cons: Blocking, coordinator single point of failure
    - Solution 2: Saga pattern
        - Draw: Chain of transactions with compensating actions
        - Pros: Non-blocking, Cons: eventual consistency
    - Which would you choose for FSL? Why?
    - Save design
- [ ] **Excalidraw:** Event-driven architecture (30 min)
    - Scenario: Appointment status changes need to notify multiple services
    - Draw: Event Bus (Kafka) architecture
    - Show: Publishers, Topics, Consumers
    - Show: Dead letter queue for failed events
    - How to ensure exactly-once delivery?
    - Save design

**Create 4 Flashcards:**

1. Q: Two-phase commit vs Saga? A: 2PC blocks until all agree (strong consistency, blocking). Saga uses compensating transactions (eventually consistent, non-blocking).
2. Q: Event sourcing? A: Store events (not current state). Benefits: audit trail, replay. Challenges: event schema evolution, storage.
3. Q: Idempotency? A: Operation can be applied multiple times with same result. Critical for retries in distributed systems.
4. Q: At-least-once vs Exactly-once? A: At-least-once simpler (duplicates possible). Exactly-once harder (needs deduplication, transactional guarantees).

---

### Friday - System Design Practice (2 hours)

**Morning Study (30 min)**

- [ ] **Review:** Your System Design Template (memorize this!)
- [ ] **Watch:** Gaurav Sen - "How to approach system design" (15 min)
    - Link: https://www.youtube.com/watch?v=0163cssUxLA
- [ ] **Review:** All Excalidraw designs from this week (15 min)
    - Can you explain each decision?

**THE SYSTEM DESIGN TEMPLATE (Use Every Time):**

```
1. REQUIREMENTS CLARIFICATION (5 minutes)
   Functional:
   - What features exactly?
   - What's in scope vs out of scope?
   
   Non-Functional:
   - How many users?
   - Requests per second?
   - Data size?
   - Latency requirements?
   - Availability requirements (99.9%? 99.99%?)

2. CAPACITY ESTIMATION (5 minutes)
   - Traffic: X requests/sec
   - Storage: Y GB/day → Z TB total
   - Bandwidth: A Mbps
   - Memory (cache): B GB

3. HIGH-LEVEL DESIGN (10 minutes)
   - Draw major components (boxes)
   - Show data flow (arrows)
   - Define APIs (REST endpoints)
   - Database schema (high-level)

4. DETAILED DESIGN (20 minutes)
   - Database schema (detailed)
   - Caching strategy
   - How to scale each component?
   - Load balancing approach
   - Handle failures?

5. BOTTLENECKS & TRADE-OFFS (5 minutes)
   - What breaks first at 10x load?
   - Consistency vs Availability trade-offs
   - Cost considerations
   - Monitoring strategy
```

**Evening Practice (1.5 hours)**

- [ ] **Full Design in Excalidraw:** Instagram/Twitter Feed (45 min)
    - Set timer for 45 minutes
    - Use template above
    - Design: Post creation, Feed generation, Follow system
    - Draw architecture in Excalidraw
    - Talk out loud (record yourself if possible)
    - Save design
- [ ] **Self-review:** (20 min)
    - What did you miss?
    - Check against System Design Primer topics
    - Read relevant sections for gaps
- [ ] **Redo weak areas:** (25 min)
    - Identify your weakest part (caching? sharding? APIs?)
    - Re-read that System Design Primer section
    - Watch relevant Gaurav Sen video

---

### 🎯 Saturday System Design Deep Dive (5 hours)

**Setup (5 min):**

- [ ] Open Excalidraw: https://excalidraw.com/
- [ ] Have System Design Primer open in another tab
- [ ] Have paper/markdown for design docs
- [ ] Set timers for each exercise

**Morning (3 hours): Three Complete Designs**

**Design #1: Real-Time Dispatch Console (60 min)**

```
REQUIREMENTS:
- 1000 dispatchers viewing live updates simultaneously
- 10K service appointments updating status per minute
- <100ms latency for updates to reach dispatchers
- Show technician real-time locations on map
- Handle dispatcher actions (assign, reassign, cancel)

YOUR PROCESS:
1. Requirements Clarification (5 min)
   - Write down: functional reqs, non-functional reqs
   
2. Capacity Estimation (5 min)
   - 10K updates/min = 167 updates/sec
   - 1000 concurrent connections
   - Estimate bandwidth
   
3. High-Level Design in Excalidraw (15 min)
   - WebSocket server for real-time updates
   - Database (which type?)
   - Caching layer (Redis?)
   - Load balancer
   - Draw and save
   
4. Detailed Design Document (20 min)
   - How does WebSocket scaling work?
   - Database schema for appointments
   - Caching strategy (what to cache?)
   - API endpoints
   
5. Scaling Strategy (10 min)
   - How to scale to 10K dispatchers?
   - What breaks first?
   - Monitoring approach
   
6. Compare to Real Systems (5 min)
   - Watch: Gaurav Sen "Whatsapp System Design" (real-time patterns)
   - Link: https://www.youtube.com/watch?v=vvhC64hQZMk
   - Note similarities

SAVE YOUR EXCALIDRAW LINK!
```

**Design #2: Geolocation Tracking System (60 min)**

```
REQUIREMENTS:
- Track 100K mobile workers' locations
- Workers send location update every 30 seconds
- Query: "Find all workers within 5km of location X"
- Privacy: PII handling (worker locations are sensitive)
- Historical data: Keep 30 days of location history

YOUR PROCESS:
1. Requirements Clarification (5 min)
   
2. Capacity Estimation (5 min)
   - 100K workers × 30sec = 3,333 updates/sec
   - Storage: 100K × 2 updates/min × 60 min × 24 hr × 30 days
   
3. High-Level Design in Excalidraw (15 min)
   - API Gateway
   - Location service
   - Database choice (PostGIS? Cassandra with geohashing?)
   - Caching (Redis with geospatial indexes?)
   - Draw and save
   
4. Detailed Design Document (20 min)
   - Geohashing or QuadTree for proximity queries?
   - Database schema
   - How to handle 3K writes/sec?
   - Privacy controls (encrypt locations? access control?)
   - Query optimization
   
5. Scaling & Privacy (10 min)
   - Shard by geographic region?
   - Privacy: GDPR considerations
   - Data retention policy
   
6. Study Real System (5 min)
   - Read: Uber Engineering Blog on geolocation
   - Link: https://www.uber.com/blog/engineering/
   - Search: "geolocation" or "maps"

SAVE YOUR EXCALIDRAW LINK!
```

**Design #3: Multi-Tenant SaaS Appointment Scheduler (60 min)**

```
REQUIREMENTS:
- 10K organizations (tenants) using the system
- 1M appointments created per day across all tenants
- Tenant isolation (each org's data must be isolated)
- 99.95% availability (only 22 minutes downtime/month)
- Role-based access control within each organization
- Per-tenant customization (custom fields, workflows)

YOUR PROCESS:
1. Requirements Clarification (5 min)
   
2. Capacity Estimation (5 min)
   - 1M appointments/day = 12 appointments/sec avg
   - Storage: 1M × 365 days = 365M appointments/year
   - Some tenants small (10 appointments/day), some huge (100K/day)
   
3. High-Level Design in Excalidraw (15 min)
   - Multi-tenancy strategy:
     * Option 1: Separate database per tenant
     * Option 2: Shared database with tenant_id in every table
     * Option 3: Shared database, separate schema per tenant
   - Which did you choose? Why?
   - Draw and save
   
4. Detailed Design Document (20 min)
   - Database schema (how to enforce tenant isolation?)
   - How to prevent tenant A from accessing tenant B's data?
   - Resource quotas (limit appointments per tenant)
   - Custom fields (how to store tenant-specific fields?)
   - Authentication/Authorization
   - API design (/api/v1/{tenantId}/appointments)
   
5. Reliability & Monitoring (10 min)
   - How to achieve 99.95% availability?
   - What if a huge tenant causes issues?
   - Circuit breaker per tenant?
   - Monitoring strategy
   
6. Learn from Salesforce (5 min)
   - Read: Salesforce Multi-Tenant Architecture
   - Link: https://developer.salesforce.com/wiki/multi_tenant_architecture
   - Note: Universal Data Dictionary, pivot tables, metadata-driven

SAVE YOUR EXCALIDRAW LINK!
```

**Afternoon (2 hours): Study Real Architectures**

**Case Study 1: Uber (30 min)**

- [ ] Watch: Gaurav Sen - "Uber System Design" (20 min)
    - Link: https://www.youtube.com/watch?v=umWABit-wbk
    - Note: Real-time matching algorithm
    - Note: Geolocation strategies
    - Note: Pricing surge algorithms
- [ ] Read: Uber Engineering Blog - Pick any article (10 min)
    - Link: https://www.uber.com/blog/engineering/
    - Look for: microservices, real-time systems, scaling

**Case Study 2: Netflix (30 min)**

- [ ] Watch: ByteByteGo - "Netflix System Design" (15 min)
    - Link: https://www.youtube.com/watch?v=psQzyFfsUGU
    - Note: CDN usage
    - Note: Caching at multiple layers
    - Note: Microservices architecture (100s of services)
- [ ] Read: Netflix Tech Blog - Caching article (15 min)
    - Link: https://netflixtechblog.com/
    - Search: "caching" or "EVCache"

**Case Study 3: Airbnb (30 min)**

- [ ] Read: Airbnb Engineering Blog - Search/Availability (20 min)
    - Link: https://medium.com/airbnb-engineering
    - Search: "search" or "availability"
    - Note: How they handle booking conflicts
- [ ] Watch: Any Airbnb system design video (10 min)
    - YouTube search: "Airbnb system design"
    - Note: Payment processing, search ranking

**Case Study 4: Salesforce (30 min)**

- [ ] Read: Salesforce Multi-Tenant Architecture (15 min)
    - Link: https://developer.salesforce.com/wiki/multi_tenant_architecture
    - Understand: How 100K+ customers share infrastructure
    - Note: Metadata-driven approach
    - Note: Universal Data Dictionary concept
- [ ] Read: Salesforce Engineering Blog (15 min)
    - Link: https://engineering.salesforce.com/
    - Pick any recent article
    - Note: How they handle scale

**Synthesis (30 min):**

- [ ] Create: "System Design Patterns Cheat Sheet" (1 page)
    
    ```
    COMMON PATTERNS ACROSS ALL SYSTEMS:
    
    1. Caching everywhere (Netflix, Uber)
    2. Microservices for scalability (all four)
    3. Sharding for large databases (Salesforce, Airbnb)
    4. Event-driven for real-time (Uber)
    5. CDN for global distribution (Netflix)
    6. Multi-tenancy strategies (Salesforce)
    
    WHAT I'D REUSE FOR FSL:
    - Uber's geolocation approach for technician tracking
    - Salesforce's multi-tenancy for different orgs
    - Netflix's caching layers for read-heavy operations
    - Event-driven for appointment status updates
    ```
    
- [ ] Save this cheat sheet - use it in interviews!
    

---

### 🎯 Sunday Mock System Design Interviews (4 hours)

**Mock Interview #1: Design Scheduling System (60 min)**

**Problem:** Design appointment scheduling system for 10K service technicians

**Setup:**

- [ ] Set timer for 45 minutes
- [ ] Open Excalidraw
- [ ] Have paper for notes
- [ ] Record yourself (audio) if possible

**Requirements (you need to ask for these):**

- How many technicians? 10K
- How many appointments/day? 100K
- Real-time scheduling or batch? Real-time preferred
- Geographic distribution? US-wide
- Latency requirements? <500ms for scheduling
- Availability? 99.9%

**Use The Template:**

1. Clarify requirements (5 min)
2. Capacity estimation (5 min)
3. High-level design (10 min)
4. Detailed design (20 min)
5. Bottlenecks (5 min)

**IMPORTANT: Talk out loud the entire time!**

**Self-Evaluation Checklist (15 min):**

- [ ] Did you ask clarifying questions?
- [ ] Did you estimate capacity (QPS, storage)?
- [ ] Did you draw clear architecture diagram?
- [ ] Did you discuss multiple approaches and choose one?
- [ ] Did you explain trade-offs?
- [ ] Did you identify bottlenecks?
- [ ] Did you talk about monitoring?
- [ ] Did you stay within 45 minutes?

**What went well:** What needs improvement:** **Next time I will:**

---

**Mock Interview #2: Design Notification Service (60 min)**

**Problem:** Design notification system for appointment updates

- 1M notifications/day (email, SMS, push)
- Delivery guarantees needed
- Priority handling (urgent vs normal)
- Rate limiting per user (don't spam)

**Setup:** Same as Mock #1

**Use The Template (45 min)**

**Self-Evaluation (15 min):** Same checklist as Mock #1

---

**Afternoon Study (2 hours):**

**Review Both Mocks (30 min):**

- [ ] Compare your designs to System Design Primer patterns
- [ ] What did you miss?
- [ ] Which sections of Primer should you re-read?

**List 5 Things to Improve (15 min):**

1. ---
    
2. ---
    
3. ---
    
4. ---
    
5. ---
    

**Deep Study Weakest Area (60 min):**

- Identify your weakest topic (caching? sharding? APIs?)
- Re-read that entire System Design Primer section
- Watch 2-3 videos on that topic
- Do a small Excalidraw exercise on that topic

**Update Flashcards (15 min):**

- Add new cards from today's learning
- Review all Week 3 cards in Anki
- Note which concepts you're still fuzzy on

---

### 📊 Week 3 Checkpoint

**You should be able to:**

- ✅ Design a scalable system in 45 minutes using Excalidraw
- ✅ Make and justify architectural trade-offs
- ✅ Estimate capacity accurately (QPS, storage, bandwidth)
- ✅ Explain CAP theorem with real examples
- ✅ Design systems that scale from 100 → 10M users
- ✅ Use proper terminology (load balancing, caching, sharding, etc.)

**Designs Completed:** 5+ full system designs in Excalidraw **Case Studies Analyzed:** 4 major architectures (Uber, Netflix, Airbnb, Salesforce) **Primary Resources Mastered:** System Design Primer core sections **Study Time:** ~20 hours

**Critical Self-Check:**

- Can you go from requirements to architecture in 10 minutes?
- Do you naturally think about failure modes?
- Can you explain WHY you chose each technology?

**If NO to any:** Spend extra time this week on that skill

**Your Excalidraw Designs:**

- [ ] URL Shortener (Monday)
- [ ] Load Balancer + Cache (Tuesday)
- [ ] Sharding Strategy (Wednesday)
- [ ] Distributed Transaction (Thursday)
- [ ] Twitter/Instagram Feed (Friday)
- [ ] Real-Time Dispatch Console (Saturday)
- [ ] Geolocation Tracking (Saturday)
- [ ] Multi-Tenant SaaS (Saturday)
- [ ] Scheduling System (Sunday)
- [ ] Notification Service (Sunday)

**Bookmark all your Excalidraw links - review them before real interviews!**

---

## Week 4: Microservices & API Design

### 📚 Learning Objectives

By end of week, you should be able to:

- Design microservices architecture with proper boundaries
- Create well-designed RESTful and event-driven APIs
- Handle distributed transactions and failures
- Explain service mesh concepts
- Design your MP-to-Core migration in extreme detail

[Content continues with detailed Week 4 schedule similar to above...]

---

## End of Part 1 - UPDATED

**Next:** Part 2 covers Weeks 5-6 (Databases & Performance)

**Progress Check:**

- [ ] Completed Weeks 1-4 with updated resources
- [ ] Used NeetCode 150 for all algorithm problems (~40 problems)
- [ ] Used System Design Primer + Gaurav Sen for system design
- [ ] Created 10+ Excalidraw designs
- [ ] Created 50+ Anki flashcards
- [ ] Total study time: ~76 hours

**All resources integrated and ready to use!**

---

_Last Updated: December 2024_ _Optimized for: SMTS/LMTS Interview Preparation at Salesforce_ _Primary Resources: NeetCode 150, System Design Primer, Gaurav Sen, Excalidraw_