# Complete Guide: Jenkins Multi-Branch Pipeline with Parallel Builds

## 📋 Overview
This guide will help you set up a Jenkins multi-branch pipeline that automatically builds `feature-login` and `feature-payment` branches concurrently for your payment-gateway application.

---

## 🎯 Critical Understanding: Two Types of Parallelism

### ⚠️ **IMPORTANT: The Key Concept**

There are **TWO DIFFERENT TYPES** of parallelism, and it's crucial to understand the difference:

#### **Type 1: Concurrent Branch Builds (Automatic - Handled by Jenkins)**
- **What it is:** Multiple branches building at the same time
- **Where you see it:** Jenkins Dashboard
- **How it works:** Jenkins automatically discovers branches and builds them concurrently
- **What you write:** NOTHING special in the Jenkinsfile - just create a Multi-Branch Pipeline job!

**Example:**
```
Dashboard shows:
  payment-gateway-app
    ├── feature-login #1 [Running...] ← Build #1
    └── feature-payment #1 [Running...] ← Build #2 (running at same time!)
```

#### **Type 2: Parallel Stages (Within One Build - Your Jenkinsfile)**
- **What it is:** Multiple stages running simultaneously within a single build
- **Where you see it:** Stage View of one build
- **How it works:** Using `parallel { }` block in your Jenkinsfile
- **What you write:** Parallel stages in Jenkinsfile (e.g., run Unit Tests and Integration Tests at the same time)

**Example:**
```
Inside feature-login build #1, Stage View shows:
  [1. Build] → [2. Test (in Parallel)]
                      ├── Unit Tests ────┐
                      └── Integration Tests ──┼──→ [3. Deploy]
```

---

## 🔑 The Core Concept: Generic Jenkinsfile

### **Critical Rule: One Generic Jenkinsfile for All Branches**

✅ **CORRECT Approach:**
- Create ONE generic Jenkinsfile
- Put the SAME file in BOTH `feature-login` and `feature-payment` branches
- The Jenkinsfile uses `env.BRANCH_NAME` to know which branch it's running on
- Jenkins runs this file separately for each branch

❌ **WRONG Approach:**
- Creating different Jenkinsfiles for different branches
- Having hardcoded stage names like "Build Feature-Login" and "Build Feature-Payment" in the same file
- Trying to build both features in one pipeline

### **How It Works:**

1. **Jenkins scans your repository** and finds branches with a Jenkinsfile
2. **For `feature-login` branch:**
   - Jenkins checks out ONLY the `feature-login` code
   - Runs the Jenkinsfile found in that branch
   - The Jenkinsfile sees `env.BRANCH_NAME = "feature-login"`

3. **For `feature-payment` branch:**
   - Jenkins checks out ONLY the `feature-payment` code
   - Runs the Jenkinsfile found in that branch
   - The Jenkinsfile sees `env.BRANCH_NAME = "feature-payment"`

4. **Both builds run concurrently** (Type 1 parallelism - automatic!)

---

## 📦 Prerequisites

### Required Software:
1. **Jenkins** installed and running
   - Download from: https://www.jenkins.io/download/
   - Version: 2.x or higher (LTS recommended)

2. **Git** installed
   - Download from: https://git-scm.com/downloads

3. **Java JDK** (required for Jenkins)
   - Version 8 or higher

### Required Jenkins Plugins:
1. **Pipeline** (usually pre-installed)
2. **Multi-Branch Pipeline** (usually pre-installed)
3. **Git Plugin** (usually pre-installed)
4. **Blue Ocean** (optional, for better UI)

---

## 🚀 STEP-BY-STEP INSTRUCTIONS

### **STEP 1: Prepare Your Git Repository**

#### 1.1 Navigate to Your Repository
```bash
# Open Git Bash or Command Prompt
cd C:\Users\91817\Devops\Jenkins-parallel-pipeline
```

#### 1.2 Check Current Branches
```bash
# Check what branches exist
git branch -a

# Check current branch
git status
```

#### 1.3 Add Jenkinsfile to feature-login Branch

**IMPORTANT:** Use the SAME generic Jenkinsfile in both branches!

```bash
# Switch to feature-login branch
git checkout feature-login

# Create or replace the Jenkinsfile (it should be the generic one)
# Copy the Jenkinsfile from the repository root

# Commit the Jenkinsfile
git add Jenkinsfile
git commit -m "Add generic Jenkinsfile for multi-branch pipeline"
git push origin feature-login
```

#### 1.4 Add Jenkinsfile to feature-payment Branch

```bash
# Switch to feature-payment branch (or create it)
git checkout feature-payment
# OR if branch doesn't exist: git checkout -b feature-payment

# Use the SAME generic Jenkinsfile
# Copy the Jenkinsfile from the repository root (same file!)

# Commit the Jenkinsfile
git add Jenkinsfile
git commit -m "Add generic Jenkinsfile for multi-branch pipeline"
git push origin feature-payment
```

**Key Point:** The Jenkinsfile should be IDENTICAL in both branches!

---

### **STEP 2: Configure Jenkins**

#### 2.1 Access Jenkins Dashboard
1. Open web browser
2. Navigate to: `http://localhost:8080` (or your Jenkins URL)
3. Login with your Jenkins credentials

#### 2.2 Install Required Plugins (if not installed)

1. Click **"Manage Jenkins"** (left sidebar)
2. Click **"Manage Plugins"**
3. Go to **"Available"** tab
4. Search and install:
   - ✅ **Pipeline**
   - ✅ **Multi-Branch Pipeline Plugin**
   - ✅ **Git Plugin**
   - ✅ **Blue Ocean** (optional, for better UI)
5. Click **"Install without restart"** or **"Download now and install after restart"**
6. Restart Jenkins if prompted

---

### **STEP 3: Create Multi-Branch Pipeline**

#### 3.1 Create New Item
1. Click **"New Item"** on Jenkins dashboard (top left)
2. Enter name: `payment-gateway-app`
3. Select **"Multi-branch Pipeline"**
4. Click **"OK"**

#### 3.2 Configure Branch Sources

**In the configuration page:**

##### **General Settings:**
- ✅ Check **"Display Name"** (optional): `Payment Gateway - Multi Branch Pipeline`

##### **Branch Sources Section:**
1. Click **"Add source"** dropdown
2. Select **"Git"**

3. **Configure Git Repository:**
   - **Project Repository URL:** 
     ```
     https://github.com/YOUR_USERNAME/Jenkins-parallel-pipeline.git
     ```
     OR
     ```
     file:///C:/Users/91817/Devops/Jenkins-parallel-pipeline
     ```
     *(Replace with your actual repository URL)*

4. **Credentials:**
   - If using GitHub/GitLab: Add credentials (Username/Password or SSH)
   - If using local repository: May not need credentials

5. **Behaviors (Click "Add" button):**
   - Add: **"Discover branches"**
     - Strategy: Select **"All branches"** OR **"Only branches that are not also filed as PRs"**
   - Add: **"Filter by name (with regular expression)"** (optional)
     - Include: `feature-.*` (only feature branches) OR `.*` (all branches)

##### **Build Configuration:**
- **Script Path:** `Jenkinsfile` (keep default - this is the file Jenkins looks for)

##### **Scan Multibranch Pipeline Triggers:**
- ✅ Check **"Periodically if not otherwise run"**
  - Interval: `1 hour` (or as needed)

##### **Orphaned Item Strategy:**
- ✅ Check **"Discard old items"**
  - Days to keep: `7`
  - Max # of builds to keep: `10`

#### 3.3 Save Configuration
1. Click **"Save"** at bottom of page

---

### **STEP 4: Initial Scan and Build**

#### 4.1 Trigger Initial Scan
1. After saving, Jenkins will automatically scan for branches
2. **OR** manually trigger scan:
   - On pipeline page, click **"Scan Multibranch Pipeline Now"** (left sidebar)

#### 4.2 View Discovered Branches
1. After scan completes (may take 30-60 seconds), you should see:
   - `feature-login`
   - `feature-payment`
   - (and any other branches with a Jenkinsfile)

#### 4.3 Automatic Builds
1. Jenkins will **automatically start building** each discovered branch
2. **This is Type 1 Parallelism!** - Both builds start at the same time!

#### 4.4 Manual Build Trigger (Optional)
If builds don't start automatically:
- Click on branch name (e.g., `feature-login`)
- Click **"Build Now"**

---

### **STEP 5: Verify Parallel Execution (The "Aha!" Moment)**

#### 5.1 Dashboard View - Type 1 Parallelism
**This is what you wanted to see!**

On the main dashboard for `payment-gateway-app`, you'll see:
```
payment-gateway-app
  ├── feature-login
  │   └── #1 [Running...] ← Build #1
  └── feature-payment
      └── #1 [Running...] ← Build #2 (running at same time!)
```

**Both builds running concurrently!** This is Type 1 parallelism - handled automatically by Jenkins.

#### 5.2 Stage View - Type 2 Parallelism
1. Click on any build (e.g., `feature-login #1`)
2. Click **"Stage View"** or **"Pipeline Steps"**
3. You'll see:
   ```
   [1. Build] 
       ↓
   [2. Test (in Parallel)]
       ├── Unit Tests ──────────┐
       └── Integration Tests ────┼──→ [3. Deploy]
   ```
   
   **Unit Tests and Integration Tests running side-by-side!** This is Type 2 parallelism - handled by your Jenkinsfile.

#### 5.3 Blue Ocean View (If Installed)
1. Click **"Open Blue Ocean"** on pipeline page
2. You'll see:
   - Visual representation of concurrent branch builds
   - Inside each build, parallel test stages shown side-by-side

#### 5.4 Console Output
1. Click on build number (e.g., `feature-login #1`)
2. Click **"Console Output"**
3. You'll see:
   ```
   BUILDING: App 'payment-gateway' for branch feature-login
   Running Unit Tests for feature-login...
   Running Integration Tests for feature-login...
   ```
   Notice: It only builds `feature-login`, not `feature-payment`!

---

### **STEP 6: Verify Both Types of Parallelism**

#### 6.1 Check Build Times

**For Type 1 (Concurrent Branch Builds):**
- Both `feature-login` and `feature-payment` builds should start within seconds of each other
- They run independently and concurrently

**For Type 2 (Parallel Stages):**
- Inside each build, Unit Tests (10s) and Integration Tests (12s) run simultaneously
- Total time for Test stage ≈ 12 seconds (not 22 seconds)
- This proves parallel execution within the build

#### 6.2 Check Console Logs

**For feature-login build:**
- Should show: `BUILDING: App 'payment-gateway' for branch feature-login`
- Should NOT mention `feature-payment`

**For feature-payment build:**
- Should show: `BUILDING: App 'payment-gateway' for branch feature-payment`
- Should NOT mention `feature-login`

---

## 🔍 Understanding the Jenkinsfile Structure

### Key Components Explained:

#### 1. **Pipeline Block**
```groovy
pipeline {
    agent any  // Runs on any available agent
}
```
- Main pipeline declaration
- `agent any`: Uses any available Jenkins agent/node

#### 2. **Generic Build Stage**
```groovy
stage('1. Build') {
    steps {
        echo "BUILDING: App 'payment-gateway' for branch ${env.BRANCH_NAME}"
    }
}
```
- **Generic:** Works for any branch
- `${env.BRANCH_NAME}`: Automatically set by Jenkins to current branch
- When running on `feature-login`, it shows "feature-login"
- When running on `feature-payment`, it shows "feature-payment"

#### 3. **Parallel Stages (Type 2)**
```groovy
stage('2. Test (in Parallel)') {
    parallel {
        stage('Unit Tests') { ... }
        stage('Integration Tests') { ... }
    }
}
```
- **This creates parallel stages within one build**
- Both test stages execute simultaneously
- Reduces total build time for that branch

#### 4. **Environment Variables**
- `${env.BRANCH_NAME}`: Current branch name (set automatically by Jenkins)
- `${env.BUILD_NUMBER}`: Build number (set automatically by Jenkins)

#### 5. **Post Actions**
- `always`: Runs after every build
- `success`: Runs only on successful builds
- `failure`: Runs only on failed builds

---

## 🎨 Customization Options

### Replace Sleep Commands with Real Build Commands

**For Node.js:**
```groovy
stage('1. Build') {
    steps {
        echo "BUILDING: App 'payment-gateway' for branch ${env.BRANCH_NAME}"
        sh 'npm install'
        sh 'npm run build'
    }
}
```

**For Python:**
```groovy
stage('1. Build') {
    steps {
        echo "BUILDING: App 'payment-gateway' for branch ${env.BRANCH_NAME}"
        sh 'pip install -r requirements.txt'
        sh 'python setup.py build'
    }
}
```

**For Java:**
```groovy
stage('1. Build') {
    steps {
        echo "BUILDING: App 'payment-gateway' for branch ${env.BRANCH_NAME}"
        sh 'mvn clean install'
    }
}
```

**For Docker:**
```groovy
stage('1. Build') {
    steps {
        echo "BUILDING: App 'payment-gateway' for branch ${env.BRANCH_NAME}"
        sh 'docker build -t payment-gateway:${env.BRANCH_NAME} .'
    }
}
```

### Add More Parallel Test Stages
```groovy
stage('2. Test (in Parallel)') {
    parallel {
        stage('Unit Tests') { ... }
        stage('Integration Tests') { ... }
        stage('Code Quality') { ... }  // Add more parallel stages
        stage('Security Scan') { ... }
    }
}
```

---

## 🐛 Troubleshooting

### Issue 1: Branches Not Discovered
**Symptoms:** Only one branch appears or no branches found

**Solutions:**
- Check repository URL is correct
- Verify credentials if using private repo
- Check branch discovery settings in configuration
- Manually trigger scan: "Scan Multibranch Pipeline Now"
- Ensure Jenkinsfile exists in the branches you want to build

### Issue 2: Builds Not Running in Parallel (Type 1)
**Symptoms:** Branches build one after another, not simultaneously

**Solutions:**
- Check number of Jenkins executors: Manage Jenkins → Manage Nodes → Configure
- Increase executors if you have only 1
- Verify both branches have a Jenkinsfile
- Check if builds are queued (look for "Waiting for next available executor")

### Issue 3: Stages Not Running in Parallel (Type 2)
**Symptoms:** Unit Tests and Integration Tests run sequentially

**Solutions:**
- Verify stages are inside `parallel { }` block
- Check Jenkinsfile syntax
- Ensure you have at least 2 executors (for 2 parallel stages)

### Issue 4: Wrong Branch Name in Logs
**Symptoms:** Build shows wrong branch name

**Solutions:**
- Verify you're using `${env.BRANCH_NAME}` (not hardcoded names)
- Check which branch Jenkins actually checked out (in console output)

### Issue 5: Jenkinsfile Not Found
**Symptoms:** Build fails with "Jenkinsfile not found"

**Solutions:**
- Ensure Jenkinsfile exists in root of repository
- Check Script Path in pipeline configuration (should be `Jenkinsfile`)
- Verify Jenkinsfile is committed to Git
- Check file permissions

### Issue 6: Permission Errors
**Solutions:**
- Check Jenkins user has file system permissions
- Verify Git credentials are correct
- Check Jenkins workspace permissions
- On Windows: Ensure Jenkins service runs with appropriate permissions

---

## 📊 Expected Results

### Success Indicators:

#### Type 1 Parallelism (Dashboard):
✅ Two separate pipeline entries for `feature-login` and `feature-payment`  
✅ Both builds start within seconds of each other  
✅ Both builds execute independently and concurrently  

#### Type 2 Parallelism (Stage View):
✅ Inside each build, test stages run in parallel  
✅ Stage View shows Unit Tests and Integration Tests side-by-side  
✅ Total test time ≈ 12 seconds (not 22 seconds)  

### Dashboard View:
```
payment-gateway-app
├── feature-login #1 [Success ✅]
└── feature-payment #1 [Success ✅]
```

Both builds should show "Success" after completion.

### Stage View (inside feature-login build):
```
[1. Build] (5 seconds)
    ↓
[2. Test (in Parallel)] (12 seconds - both tests run together)
    ├── Unit Tests ──────────┐
    └── Integration Tests ────┼──→ [3. Deploy] (3 seconds)
```

**Total build time:** ~20 seconds (not 30+ if sequential)

---

## 🎓 Next Steps

1. **Add Real Build Commands:** Replace `sleep` with actual build commands for your tech stack
2. **Add Notifications:** Configure email/Slack notifications on build completion
3. **Add Artifacts:** Store build artifacts for deployment
4. **Add Deployment Stages:** Automate deployment to different environments based on branch
5. **Add Quality Gates:** Integrate code quality checks (SonarQube, etc.)
6. **Add Docker:** Containerize your builds for consistency
7. **Conditional Logic:** Use `when` clauses to run different stages based on branch

---

## 📝 Summary Checklist

- [ ] Jenkins installed and running
- [ ] Required plugins installed (Pipeline, Multi-Branch Pipeline, Git)
- [ ] Git repository with `feature-login` and `feature-payment` branches
- [ ] **Generic Jenkinsfile exists in BOTH branches (same file!)**
- [ ] Multi-branch pipeline created in Jenkins
- [ ] Branch sources configured with repository URL
- [ ] Initial scan completed
- [ ] Both branches discovered and built
- [ ] **Type 1 Parallelism verified:** Both builds run concurrently on dashboard
- [ ] **Type 2 Parallelism verified:** Parallel stages visible in Stage View
- [ ] Console output shows correct branch names

---

## 💡 Key Takeaways

1. **Multi-Branch Pipeline** automatically discovers and builds all branches that have a Jenkinsfile
2. **Type 1 Parallelism (Concurrent Branch Builds):** Automatic - just create a Multi-Branch Pipeline job!
3. **Type 2 Parallelism (Parallel Stages):** Use `parallel { }` block in your Jenkinsfile
4. **Generic Jenkinsfile:** ONE file for ALL branches - uses `env.BRANCH_NAME` to adapt
5. **Dashboard shows concurrent builds** - this is what you wanted!
6. **Each branch builds independently** - Jenkins checks out only that branch's code

---

## 🆘 Need Help?

Common commands for verification:
```bash
# Check Git branches
git branch -a

# Verify Jenkinsfile exists in both branches
git checkout feature-login
ls -la Jenkinsfile

git checkout feature-payment
ls -la Jenkinsfile

# Check Jenkins logs (Windows)
# Navigate to Jenkins installation directory
# Check: jenkins\logs\jenkins.log

# Validate Jenkinsfile syntax in Jenkins:
# Dashboard → Your Pipeline → Pipeline Syntax → Validate Jenkinsfile
```

---

**Congratulations! 🎉 You've successfully set up a Jenkins Multi-Branch Pipeline with both types of parallelism!**

You'll now see:
- ✅ **Concurrent builds** on the dashboard (Type 1)
- ✅ **Parallel stages** within each build (Type 2)
