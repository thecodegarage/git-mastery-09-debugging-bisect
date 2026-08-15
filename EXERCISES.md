# Debugging & Bisect - Exercises 🔍

12 hands-on exercises to master Git debugging and bug-hunting tools.

**⚠️ IMPORTANT**: Run `./build-history.sh` before starting exercises!

---

## 🟢 Git Blame Basics (Exercises 1-3)

### Exercise 1: Basic Git Blame

**Objective**: Track who changed specific lines of code

**Scenario**: You found a bug and need to know who wrote the problematic code.

**Tasks**:

1. Check recent commits:
   ```bash
   git log --oneline -10
   ```

2. View current calculator code:
   ```bash
   cat src/calc.js
   ```

3. Use git blame to see line-by-line authorship:
   ```bash
   git blame src/calc.js
   ```

4. Blame specific lines only:
   ```bash
   git blame -L 10,15 src/calc.js
   # Shows lines 10-15 only
   ```

5. Show email addresses:
   ```bash
   git blame -e src/calc.js
   ```

6. Ignore whitespace changes:
   ```bash
   git blame -w src/calc.js
   ```

**Validation**:
```bash
# Each line shows: commit author date line-number content
# Example: a1b2c3d4 (Bug Hunter 2024-01-15 14:00:00 1) function add(a, b) {
```

**Learning Points**:
- ✅ `git blame` shows who last modified each line
- ✅ Shows commit SHA, author, date, and line content
- ✅ Use `-L` to focus on specific lines
- ✅ `-e` shows email, `-w` ignores whitespace

---

### Exercise 2: Blame with Line History

**Objective**: Track how a line evolved over time

**Scenario**: A line has been modified multiple times. Track its full history.

**Tasks**:

1. Blame with commit messages:
   ```bash
   git blame -s src/calc.js
   # -s shows short commit hash only
   ```

2. Follow line history across renames:
   ```bash
   git blame -C src/calc.js
   # Detects code moved from other files
   ```

3. Follow line history across file copies:
   ```bash
   git blame -C -C src/calc.js
   # Even more aggressive detection
   ```

4. Show line history through time:
   ```bash
   # Find commit where line was added
   git log -S "function divide" --oneline
   ```

5. Blame at specific commit:
   ```bash
   git blame <commit-sha> -- src/calc.js
   # Shows blame before that commit
   ```

6. Combine with log for full story:
   ```bash
   # Get commit from blame
   COMMIT=$(git blame -L 13,13 src/calc.js | awk '{print $1}')
   git show $COMMIT
   ```

**Validation**:
```bash
# Should show commit that introduced the divide function
git log --oneline --all | grep -i divide
```

**Learning Points**:
- ✅ `-C` detects moved/copied code
- ✅ `git log -S` finds when text was added/removed
- ✅ Blame works at any point in history
- ✅ Combine blame + show for full context

---

### Exercise 3: Interactive Blame Investigation

**Objective**: Use blame to investigate the division bug

**Scenario**: Division function returns wrong results. Find who introduced the bug.

**Tasks**:

1. Identify the buggy line:
   ```bash
   git blame src/calc.js | grep -A2 "function divide"
   ```

2. Get the commit SHA:
   ```bash
   git blame -L 13,17 src/calc.js
   # Look for the divide function implementation
   ```

3. View that commit:
   ```bash
   git show <commit-sha-from-blame>
   ```

4. See what else changed in that commit:
   ```bash
   git show --stat <commit-sha>
   ```

5. Check who authored it:
   ```bash
   git log --format="%H %an %ae %s" | grep <commit-sha>
   ```

6. View the file before the bug:
   ```bash
   git show <commit-sha>~1:src/calc.js
   # ~1 means parent commit
   ```

**Validation**:
```bash
# The buggy line should be: return a * b;
# Should be: return a / b;
git blame src/calc.js | grep "return a \* b"
```

**Learning Points**:
- ✅ Blame identifies exact commit introducing code
- ✅ Use `git show` to see full commit context
- ✅ `~1` syntax shows parent commit
- ✅ Blame + show = complete bug history

---

## 🟡 Git Bisect Basics (Exercises 4-7)

### Exercise 4: Manual Bisect

**Objective**: Use git bisect to find when bug was introduced

**Scenario**: Division works incorrectly. Find the exact commit that broke it.

**Tasks**:

1. Start bisect:
   ```bash
   git bisect start
   ```

2. Mark current commit as bad:
   ```bash
   git bisect bad
   # Current HEAD has the bug
   ```

3. Mark an early commit as good:
   ```bash
   git log --oneline
   # Pick a commit before commit 25 (e.g., commit 10)
   git bisect good <commit-sha>
   ```

4. Git checks out middle commit:
   ```bash
   # Git automatically checks out midpoint
   cat src/calc.js
   # Test if divide function works correctly
   ```

5. Test the code:
   ```bash
   # If divide exists and works:
   git bisect good
   
   # If bug exists:
   git bisect bad
   
   # If divide doesn't exist yet:
   git bisect skip
   ```

6. Continue until Git finds the bad commit:
   ```bash
   # Keep marking good/bad until Git reports:
   # "<sha> is the first bad commit"
   ```

7. View the bad commit:
   ```bash
   git show
   ```

8. Exit bisect:
   ```bash
   git bisect reset
   # Returns to original HEAD
   ```

**Validation**:
```bash
# Bisect should identify commit 25 as the bad commit
git log --oneline | sed -n '16p'
# Should show "Add division feature"
```

**Learning Points**:
- ✅ Bisect uses binary search (O(log n))
- ✅ Mark commits as good/bad/skip
- ✅ Git automatically finds culprit
- ✅ Always `git bisect reset` when done

---

### Exercise 5: Bisect with Terms

**Objective**: Use custom terms instead of good/bad

**Scenario**: Find when a feature was added (not a bug).

**Tasks**:

1. Start bisect with custom terms:
   ```bash
   git bisect start --term-old=without --term-new=with
   ```

2. Mark current commit (has feature):
   ```bash
   git bisect with
   ```

3. Mark old commit (no feature):
   ```bash
   git bisect without <old-commit>
   ```

4. Test each commit:
   ```bash
   # If feature exists:
   git bisect with
   
   # If feature missing:
   git bisect without
   ```

5. Find first commit with feature:
   ```bash
   # Git reports first commit with the feature
   ```

6. Reset:
   ```bash
   git bisect reset
   ```

**Validation**:
```bash
# Should find commit where divide was added
git log --oneline --all -S "function divide"
```

**Learning Points**:
- ✅ Use `--term-old` and `--term-new` for clarity
- ✅ Not just for bugs - find any change
- ✅ Makes bisect logs more readable
- ✅ Custom terms: broken/fixed, slow/fast, etc.

---

### Exercise 6: Automated Bisect with Script

**Objective**: Automate bisect with a test script

**Scenario**: Write script to automatically test each commit.

**Tasks**:

1. Create test script:
   ```bash
   cat > test-divide.sh << 'EOF'
#!/bin/bash
# Test if divide function works correctly

if ! grep -q "function divide" src/calc.js; then
    # Function doesn't exist yet - skip
    exit 125
fi

if grep -q "return a \* b;" src/calc.js | grep -A1 "function divide"; then
    # Bug exists: divide returns a * b
    exit 1
fi

# Bug doesn't exist: divide works correctly
exit 0
EOF
   chmod +x test-divide.sh
   ```

2. Run automated bisect:
   ```bash
   git bisect start HEAD <good-commit>
   git bisect run ./test-divide.sh
   ```

3. Git automatically tests all commits:
   ```bash
   # Git runs script on each commit
   # Exit 0 = good
   # Exit 1 = bad
   # Exit 125 = skip
   ```

4. View results:
   ```bash
   # Git shows first bad commit
   git show
   ```

5. Clean up:
   ```bash
   git bisect reset
   rm test-divide.sh
   ```

**Validation**:
```bash
# Should identify commit 25 automatically
# Much faster than manual testing
```

**Learning Points**:
- ✅ `git bisect run <script>` automates testing
- ✅ Exit codes: 0 (good), 1-127 except 125 (bad), 125 (skip)
- ✅ Works with any test: scripts, unit tests, compilation
- ✅ Saves time on large histories

---

### Exercise 7: Bisect with Test Suite

**Objective**: Use existing test suite for bisect

**Scenario**: Project has automated tests. Use them for bisect.

**Tasks**:

1. Create simple test file:
   ```bash
   cat > test-calc.js << 'EOF'
const calc = require('./src/calc.js');

// Test division
if (calc.divide) {
    const result = calc.divide(10, 2);
    if (result !== 5) {
        console.error('FAIL: Division broken');
        process.exit(1);
    }
}

console.log('PASS: All tests pass');
process.exit(0);
EOF
   ```

2. Test the script manually:
   ```bash
   node test-calc.js
   ```

3. Use with bisect:
   ```bash
   git bisect start HEAD <good-commit>
   git bisect run node test-calc.js
   ```

4. Let Git find the bug:
   ```bash
   # Git automatically runs test at each commit
   ```

5. Clean up:
   ```bash
   git bisect reset
   rm test-calc.js
   ```

**Validation**:
```bash
# Should find commit 25 where divide bug was introduced
```

**Learning Points**:
- ✅ Real test suites work with bisect
- ✅ Exit codes matter: 0 (pass), non-zero (fail)
- ✅ Bisect integrates with CI/CD tests
- ✅ Automate regression testing

---

## 🔵 Advanced Debugging (Exercises 8-10)

### Exercise 8: Git Log Search

**Objective**: Search commit messages and code for clues

**Scenario**: Find all commits related to a feature.

**Tasks**:

1. Search commit messages:
   ```bash
   git log --oneline --all --grep="divide"
   # Finds commits with "divide" in message
   ```

2. Case-insensitive search:
   ```bash
   git log --oneline --all --grep="DIVIDE" -i
   ```

3. Search for code changes:
   ```bash
   git log -S "function divide" --oneline
   # Finds commits that added/removed this text
   ```

4. Search with regex:
   ```bash
   git log -G "function (add|subtract|multiply|divide)" --oneline
   ```

5. Show patches with search:
   ```bash
   git log -S "function divide" -p
   # Shows full diff for matching commits
   ```

6. Search by author:
   ```bash
   git log --author="Bug Hunter" --oneline
   ```

7. Search by date:
   ```bash
   git log --since="2024-01-10" --until="2024-01-20" --oneline
   ```

8. Combine searches:
   ```bash
   git log --author="Bug Hunter" --grep="divide" --oneline
   ```

**Validation**:
```bash
# Should find commit 25 with "Add division feature"
git log --oneline | grep -i division
```

**Learning Points**:
- ✅ `--grep` searches commit messages
- ✅ `-S` searches code content (pickaxe)
- ✅ `-G` uses regex patterns
- ✅ Combine filters for precise searches

---

### Exercise 9: Git Grep for Code Search

**Objective**: Search current codebase for patterns

**Scenario**: Find all functions or specific code patterns.

**Tasks**:

1. Search for text in tracked files:
   ```bash
   git grep "function"
   ```

2. Show line numbers:
   ```bash
   git grep -n "function"
   ```

3. Count matches:
   ```bash
   git grep -c "function"
   # Shows count per file
   ```

4. Search with context:
   ```bash
   git grep -A 3 -B 1 "function divide"
   # -A = after, -B = before
   ```

5. Case-insensitive:
   ```bash
   git grep -i "FUNCTION"
   ```

6. Search at specific commit:
   ```bash
   git grep "function divide" <commit-sha>
   ```

7. Search with regex:
   ```bash
   git grep -E "function (add|subtract|multiply|divide)"
   ```

8. Show function names:
   ```bash
   git grep --show-function "return a"
   ```

**Validation**:
```bash
# Find all function definitions
git grep -n "^function" src/
```

**Learning Points**:
- ✅ `git grep` searches working tree and history
- ✅ Faster than regular grep (uses Git index)
- ✅ Works at any commit with `git grep <sha>`
- ✅ Respects .gitignore automatically

---

### Exercise 10: Diff and Patch Analysis

**Objective**: Analyze changes between commits

**Scenario**: Compare versions to understand changes.

**Tasks**:

1. Diff between commits:
   ```bash
   git diff <old-commit> <new-commit>
   ```

2. Diff specific file:
   ```bash
   git diff <commit1> <commit2> -- src/calc.js
   ```

3. Show word-level diff:
   ```bash
   git diff --word-diff <commit1> <commit2>
   ```

4. Ignore whitespace:
   ```bash
   git diff -w <commit1> <commit2>
   ```

5. Show stat summary:
   ```bash
   git diff --stat <commit1> <commit2>
   ```

6. Show only changed function names:
   ```bash
   git diff --function-context <commit1> <commit2>
   ```

7. Create patch file:
   ```bash
   git diff <commit1> <commit2> > changes.patch
   ```

8. Apply patch:
   ```bash
   git apply changes.patch
   ```

**Validation**:
```bash
# Compare before/after bug introduction
git diff HEAD~15 HEAD -- src/calc.js
```

**Learning Points**:
- ✅ `git diff` compares any two commits
- ✅ Many options: word-diff, stat, function-context
- ✅ Create patches for sharing changes
- ✅ Use `--` to separate commits from files

---

## 🟣 Real-World Debugging (Exercises 11-12)

### Exercise 11: Debugging Workflow

**Objective**: Complete debugging workflow from symptoms to fix

**Scenario**: Bug reported. Find and fix it systematically.

**Tasks**:

1. Reproduce the bug:
   ```bash
   # Bug: divide(10, 2) returns 20 instead of 5
   git log --oneline -5
   ```

2. Use blame to find suspect:
   ```bash
   git blame src/calc.js | grep -A2 "function divide"
   ```

3. View suspect commit:
   ```bash
   COMMIT=$(git blame src/calc.js | grep "function divide" | awk '{print $1}')
   git show $COMMIT
   ```

4. Verify bug doesn't exist earlier:
   ```bash
   git checkout $COMMIT~1
   cat src/calc.js
   # divide shouldn't exist or should work correctly
   ```

5. Use bisect to confirm:
   ```bash
   git checkout master
   git bisect start HEAD $COMMIT~5
   git bisect run bash -c "grep -q 'return a \* b' src/calc.js && grep -B2 'return a \* b' src/calc.js | grep -q 'function divide'"
   ```

6. Document findings:
   ```bash
   echo "Bug introduced in commit $COMMIT" > bug-report.txt
   echo "Line: return a * b; should be return a / b;" >> bug-report.txt
   git bisect reset
   ```

7. Create fix:
   ```bash
   # Fix the bug manually
   sed -i 's/return a \* b;  \/\/ BUG/return a \/ b;/' src/calc.js
   git diff src/calc.js
   ```

8. Verify fix:
   ```bash
   # Test that divide(10, 2) now returns 5
   cat src/calc.js | grep -A3 "function divide"
   ```

**Validation**:
```bash
# Should identify commit 25 and fix the division
git diff src/calc.js | grep -A1 "function divide"
```

**Learning Points**:
- ✅ Systematic approach: reproduce → blame → show → bisect → fix
- ✅ Document findings for team
- ✅ Always verify fix before committing
- ✅ Combine multiple Git tools

---

### Exercise 12: Performance Regression Hunt

**Objective**: Find when performance degraded

**Scenario**: Code is slow. Find commit that caused slowdown.

**Tasks**:

1. Create performance test:
   ```bash
   cat > perf-test.sh << 'EOF'
#!/bin/bash
# Simulate performance test

LINES=$(wc -l < src/calc.js)

# Assume code > 30 lines is "slow"
if [ $LINES -gt 30 ]; then
    echo "SLOW: $LINES lines"
    exit 1
fi

echo "FAST: $LINES lines"
exit 0
EOF
   chmod +x perf-test.sh
   ```

2. Run bisect with performance test:
   ```bash
   git bisect start --term-old=fast --term-new=slow
   git bisect slow HEAD
   git bisect fast <early-commit>
   git bisect run ./perf-test.sh
   ```

3. Find first "slow" commit:
   ```bash
   # Git identifies where code became slow
   git show
   ```

4. Analyze the changes:
   ```bash
   git diff HEAD~1 HEAD --stat
   ```

5. Clean up:
   ```bash
   git bisect reset
   rm perf-test.sh
   ```

**Validation**:
```bash
# Should identify when code complexity increased
git log --oneline --all
```

**Learning Points**:
- ✅ Bisect works for any regression
- ✅ Performance, bugs, behavior changes
- ✅ Custom test scripts for any criteria
- ✅ Use meaningful terms (fast/slow, working/broken)

---

## 🎯 Command Reference

### Git Blame
```bash
git blame <file>                    # Show line-by-line authorship
git blame -L 10,20 <file>          # Specific lines only
git blame -e <file>                 # Show email addresses
git blame -w <file>                 # Ignore whitespace
git blame -C <file>                 # Detect moved code
git blame <commit> -- <file>        # Blame at specific commit
```

### Git Bisect
```bash
git bisect start                    # Start bisect session
git bisect bad [commit]             # Mark commit as bad
git bisect good <commit>            # Mark commit as good
git bisect skip                     # Skip untestable commit
git bisect reset                    # Exit and return to HEAD
git bisect run <script>             # Automate with script
git bisect start --term-old=<term> --term-new=<term>  # Custom terms
```

### Git Log Search
```bash
git log --grep="pattern"            # Search commit messages
git log -S "text"                   # Pickaxe: find text changes
git log -G "regex"                  # Regex pattern search
git log --author="name"             # Filter by author
git log --since="date"              # Filter by date
git log -p                          # Show patches
```

### Git Grep
```bash
git grep "pattern"                  # Search tracked files
git grep -n "pattern"               # Show line numbers
git grep -c "pattern"               # Count matches
git grep -i "pattern"               # Case-insensitive
git grep -E "regex"                 # Extended regex
git grep "pattern" <commit>         # Search at commit
```

---

**Congratulations!** You've mastered Git debugging tools! 🔍

**Key Takeaways**:
- `git blame` finds who wrote code
- `git bisect` finds when bugs were introduced
- `git log` searches history
- `git grep` searches code
- Automate bisect with scripts
- Combine tools for thorough investigation

**Next Steps**:
1. Practice on real bugs in your projects
2. Automate bisect in CI/CD
3. Take the final quiz
4. You've completed the Git Mastery course! 🎓
