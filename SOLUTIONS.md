# Debugging & Bisect - Solutions 🔍

Solutions for all debugging and bisect exercises.

---

## 🟢 Git Blame Basics Solutions

### Exercise 1: Basic Git Blame

**Solution**:
```bash
# View commit history
git log --oneline -10

# Basic blame
git blame src/calc.js

# Specific lines
git blame -L 10,15 src/calc.js

# With email
git blame -e src/calc.js

# Ignore whitespace
git blame -w src/calc.js
```

**Output Format**:
```
commit-sha (Author Name 2024-01-15 14:00:00 1) function add(a, b) {
commit-sha (Author Name 2024-01-15 14:00:00 2)     return a + b;
commit-sha (Author Name 2024-01-15 14:00:00 3) }
```

**Key Points**:
- Each line shows: SHA, author, date, line number, content
- Use `-L start,end` for specific line ranges
- `-e` shows email instead of name
- `-w` ignores whitespace-only changes

---

### Exercise 2: Blame with Line History

**Solution**:
```bash
# Short SHA only
git blame -s src/calc.js

# Detect moved code
git blame -C src/calc.js

# More aggressive detection
git blame -C -C src/calc.js

# Find when text was added
git log -S "function divide" --oneline

# Blame at specific commit
git blame <commit-sha> -- src/calc.js

# Get commit details from blame
COMMIT=$(git blame -L 13,13 src/calc.js | awk '{print $1}')
git show $COMMIT
```

**Finding Line History**:
```bash
# Step 1: Find commits that changed the text
git log -S "function divide" --oneline

# Step 2: View commit details
git show <commit-from-above>

# Step 3: See file before the change
git show <commit>~1:src/calc.js
```

---

### Exercise 3: Interactive Blame Investigation

**Solution**:
```bash
# Find buggy line
git blame src/calc.js | grep -A2 "function divide"

# Get specific line range
git blame -L 13,17 src/calc.js

# Example output:
# 25abcdef (Bug Hunter 2024-01-15 10:00:00 13) function divide(a, b) {
# 25abcdef (Bug Hunter 2024-01-15 10:00:00 14)     return a * b; // BUG!
# 25abcdef (Bug Hunter 2024-01-15 10:00:00 15) }

# View that commit
COMMIT=$(git blame -L 14,14 src/calc.js | awk '{print $1}')
git show $COMMIT

# See what else changed
git show --stat $COMMIT

# View file before bug
git show $COMMIT~1:src/calc.js

# Confirm the bug line
git blame src/calc.js | grep "return a \* b"
```

**The Bug**:
- Line: `return a * b;` in divide function
- Should be: `return a / b;`
- Introduced in commit 25

---

## 🟡 Git Bisect Basics Solutions

### Exercise 4: Manual Bisect

**Solution**:
```bash
# Start bisect
git bisect start

# Current commit is bad (has bug)
git bisect bad

# Find a good commit (before commit 25)
git log --oneline | tail -20
git bisect good <commit-sha-before-25>

# Git checks out middle commit
# Test the code:
cat src/calc.js

# If divide exists and is buggy:
git bisect bad

# If divide doesn't exist or works correctly:
git bisect good

# If divide doesn't exist yet:
git bisect skip

# Continue marking good/bad until:
# Git reports: "abc123 is the first bad commit"

# View the culprit
git show

# Exit bisect
git bisect reset
```

**Bisect Process**:
```
Total commits: 40
First iteration: test commit 20 (midpoint)
Second iteration: test commit 25 or 15 (depending on result)
...continues until bad commit found

Expected result: Commit 25 "Add division feature"
```

---

### Exercise 5: Bisect with Terms

**Solution**:
```bash
# Start with custom terms
git bisect start --term-old=without --term-new=with

# Current has feature
git bisect with

# Old commit doesn't have feature
git bisect without <old-commit>

# Test each commit Git checks out
cat src/calc.js | grep "function divide"

# If divide exists:
git bisect with

# If divide missing:
git bisect without

# Git finds first commit with divide
git show

# Reset
git bisect reset
```

**Custom Terms**:
- `without` / `with` for features
- `slow` / `fast` for performance
- `broken` / `working` for bugs
- More intuitive than good/bad

---

### Exercise 6: Automated Bisect with Script

**Solution**:
```bash
# Create test script
cat > test-divide.sh << 'EOF'
#!/bin/bash

# Check if divide function exists
if ! grep -q "function divide" src/calc.js; then
    exit 125  # Skip - feature not added yet
fi

# Check for bug (multiply instead of divide)
if grep -A2 "function divide" src/calc.js | grep -q "return a \* b;"; then
    exit 1  # Bad - bug exists
fi

exit 0  # Good - no bug
EOF

chmod +x test-divide.sh

# Test manually first
./test-divide.sh
echo $?  # Should be 0 (good) or 1 (bad) or 125 (skip)

# Run automated bisect
git bisect start HEAD <good-commit>
git bisect run ./test-divide.sh

# Git automatically tests all commits
# Reports: "abc123 is the first bad commit"

# View result
git show

# Clean up
git bisect reset
rm test-divide.sh
```

**Exit Codes**:
- `0` = Good commit (test passes)
- `1-127` (except 125) = Bad commit (test fails)
- `125` = Skip commit (can't test)

---

### Exercise 7: Bisect with Test Suite

**Solution**:
```bash
# Create test file
cat > test-calc.js << 'EOF'
const calc = require('./src/calc.js');

// Test division if it exists
if (calc.divide) {
    const result = calc.divide(10, 2);
    if (result !== 5) {
        console.error(`FAIL: Expected 5, got ${result}`);
        process.exit(1);
    }
}

console.log('PASS: All tests pass');
process.exit(0);
EOF

# Test manually
node test-calc.js

# Run bisect
git bisect start HEAD <good-commit>
git bisect run node test-calc.js

# View result
git show

# Clean up
git bisect reset
rm test-calc.js
```

**Using Real Test Suites**:
```bash
# With npm test
git bisect run npm test

# With pytest
git bisect run python -m pytest

# With custom script
git bisect run ./run-tests.sh
```

---

## 🔵 Advanced Debugging Solutions

### Exercise 8: Git Log Search

**Solution**:
```bash
# Search commit messages
git log --oneline --all --grep="divide"

# Case-insensitive
git log --oneline --all --grep="DIVIDE" -i

# Search code changes (pickaxe)
git log -S "function divide" --oneline

# Regex search
git log -G "function (add|subtract|multiply|divide)" --oneline

# Show patches
git log -S "function divide" -p

# By author
git log --author="Bug Hunter" --oneline

# By date
git log --since="2024-01-10" --until="2024-01-20" --oneline

# Combined search
git log --author="Bug Hunter" --grep="divide" --oneline
```

**Advanced Searches**:
```bash
# Find commits affecting specific function
git log -L :divide:src/calc.js

# Find commits in date range with changes
git log --since="1 week ago" --author="Alice" -S "function" --oneline

# Search multiple files
git log -- src/calc.js src/utils.js
```

---

### Exercise 9: Git Grep for Code Search

**Solution**:
```bash
# Basic search
git grep "function"

# With line numbers
git grep -n "function"

# Count matches
git grep -c "function"

# With context
git grep -A 3 -B 1 "function divide"

# Case-insensitive
git grep -i "FUNCTION"

# At specific commit
git grep "function divide" <commit-sha>

# Regex search
git grep -E "function (add|subtract|multiply|divide)"

# Show function names
git grep --show-function "return a"
```

**Useful Grep Patterns**:
```bash
# Find all function definitions
git grep -n "^function"

# Find TODOs
git grep -n "TODO\|FIXME"

# Find console.log statements
git grep -n "console\.log"

# Search in specific directory
git grep "function" -- src/

# Show only filenames
git grep -l "function"
```

---

### Exercise 10: Diff and Patch Analysis

**Solution**:
```bash
# Diff between commits
git diff <old-commit> <new-commit>

# Specific file
git diff <commit1> <commit2> -- src/calc.js

# Word-level diff
git diff --word-diff <commit1> <commit2>

# Ignore whitespace
git diff -w <commit1> <commit2>

# Stat summary
git diff --stat <commit1> <commit2>

# Function context
git diff --function-context <commit1> <commit2>

# Create patch
git diff <commit1> <commit2> > changes.patch

# Apply patch
git apply changes.patch

# Check if patch applies cleanly
git apply --check changes.patch
```

**Diff Options**:
```bash
# Show only names of changed files
git diff --name-only <commit1> <commit2>

# Show names and status (A=added, M=modified, D=deleted)
git diff --name-status <commit1> <commit2>

# Color words
git diff --color-words <commit1> <commit2>

# Histogram algorithm (better for code)
git diff --histogram <commit1> <commit2>
```

---

## 🟣 Real-World Debugging Solutions

### Exercise 11: Debugging Workflow

**Complete Workflow**:
```bash
# Step 1: Reproduce bug
echo "Bug: divide(10, 2) returns 20 instead of 5"

# Step 2: Use blame to find suspect
git blame src/calc.js | grep -A2 "function divide"
# Output shows commit 25abcdef

# Step 3: View suspect commit
COMMIT=$(git blame src/calc.js | grep "function divide" | head -1 | awk '{print $1}')
git show $COMMIT

# Step 4: Verify bug doesn't exist earlier
git checkout $COMMIT~1
cat src/calc.js  # Should not have buggy divide
git checkout master

# Step 5: Use bisect to confirm
git bisect start HEAD $COMMIT~5

# Create quick test
cat > quick-test.sh << 'EOF'
#!/bin/bash
grep -A2 "function divide" src/calc.js 2>/dev/null | grep -q "return a \* b;" && exit 1
exit 0
EOF
chmod +x quick-test.sh

git bisect run ./quick-test.sh

# Step 6: Document findings
cat > bug-report.md << EOF
# Bug Report

**Issue**: Division returns multiplication result
**Commit**: $COMMIT
**File**: src/calc.js
**Line**: return a * b; (should be return a / b;)
**Introduced**: $(git log --format="%ai" -1 $COMMIT)
**Author**: $(git log --format="%an <%ae>" -1 $COMMIT)
EOF

git bisect reset
rm quick-test.sh

# Step 7: Create fix
sed -i 's/return a \* b;/return a \/ b;/g' src/calc.js

# Step 8: Verify fix
git diff src/calc.js

# Step 9: Commit fix
git add src/calc.js
git commit -m "Fix division bug: Use division operator instead of multiplication

Closes: Bug introduced in $COMMIT
The divide function was incorrectly using '*' instead of '/'

Before: return a * b;
After:  return a / b;"
```

---

### Exercise 12: Performance Regression Hunt

**Solution**:
```bash
# Create performance test
cat > perf-test.sh << 'EOF'
#!/bin/bash

LINES=$(wc -l < src/calc.js)

# Assume > 30 lines is "slow"
if [ $LINES -gt 30 ]; then
    echo "SLOW: $LINES lines"
    exit 1
fi

echo "FAST: $LINES lines"
exit 0
EOF

chmod +x perf-test.sh

# Test manually
./perf-test.sh

# Run bisect
git bisect start --term-old=fast --term-new=slow
git bisect slow HEAD
git bisect fast <early-commit>
git bisect run ./perf-test.sh

# Find first slow commit
git show

# Analyze
git diff HEAD~1 HEAD --stat

# Clean up
git bisect reset
rm perf-test.sh
```

**Real Performance Testing**:
```bash
# With timing
cat > perf-test.sh << 'EOF'
#!/bin/bash
start=$(date +%s%N)
# Run your code/tests
node src/calc.js
end=$(date +%s%N)

duration=$(( (end - start) / 1000000 ))  # milliseconds

if [ $duration -gt 100 ]; then
    echo "SLOW: ${duration}ms"
    exit 1
fi

echo "FAST: ${duration}ms"
exit 0
EOF
```

---

## 🎯 Debugging Cheat Sheet

### Common Workflows

**Find Who Wrote Code**:
```bash
git blame <file>
git blame -L 10,20 <file>
```

**Find When Bug Introduced**:
```bash
git bisect start HEAD <good-commit>
git bisect run <test-script>
```

**Search Commit History**:
```bash
git log --grep="keyword"
git log -S "code text"
git log --author="name"
```

**Search Code**:
```bash
git grep -n "pattern"
git grep "pattern" <commit>
```

**Compare Versions**:
```bash
git diff <commit1> <commit2>
git diff --stat <commit1> <commit2>
```

### Bisect Script Template

```bash
#!/bin/bash
# Exit 0 = good
# Exit 1 = bad
# Exit 125 = skip

# Check if feature exists
if ! grep -q "feature" src/file.js; then
    exit 125  # Skip
fi

# Run test
if test-command; then
    exit 0  # Good
else
    exit 1  # Bad
fi
```

### Debugging Decision Tree

```
Found a bug?
├─ Know when it started?
│  ├─ Yes → Use git show to see changes
│  └─ No → Use git bisect to find it
│
├─ Know who wrote it?
│  ├─ Yes → Ask them directly
│  └─ No → Use git blame to find author
│
└─ Need to search history?
   ├─ Search commits → git log --grep
   ├─ Search code → git log -S or git grep
   └─ Compare versions → git diff
```

---

## 🎓 Best Practices

### Blame Best Practices
```
✅ Use blame to understand, not to blame people
✅ Check commit context with git show
✅ Use -w to ignore formatting changes
✅ Use -C to track moved code
❌ Don't assume blamed author is responsible (code may have moved)
```

### Bisect Best Practices
```
✅ Always git bisect reset when done
✅ Automate with scripts when possible
✅ Use meaningful terms (--term-old/--term-new)
✅ Mark commits as skip if untestable
❌ Don't bisect without a good "good" commit
❌ Don't forget to reset before switching tasks
```

### Search Best Practices
```
✅ Use -S (pickaxe) to find when text changed
✅ Use -G for regex patterns
✅ Use git grep for current code
✅ Use git log for history
❌ Don't confuse -S (pickaxe) and --grep (messages)
```

---

**Congratulations!** You're now a Git debugging expert! 🎉

**Key Debugging Tools**:
1. **git blame** - Find who wrote code
2. **git bisect** - Find when bugs appeared
3. **git log** - Search history
4. **git grep** - Search code
5. **git diff** - Compare versions

**Next Steps**:
1. Apply these techniques to real projects
2. Create bisect scripts for your test suites
3. Document your debugging findings
4. Share knowledge with your team
5. Take the final quiz! 🎓

**You've completed Git Mastery!** 🚀
