# 🚀 Quick Start Guide - Jenkins Multi-Branch Pipeline

## ⚡ Quick Reference (5 Minutes Setup)

### Prerequisites Checklist
- [ ] Jenkins running at http://localhost:8080
- [ ] Git repository with `feature-login` and `feature-payment` branches
- [ ] **Generic Jenkinsfile committed to BOTH branches** (same file!)

---

## 🎯 Understanding the Two Types of Parallelism

### Type 1: Concurrent Branch Builds (Automatic)
- **What:** Multiple branches building at the same time
- **Where:** Jenkins Dashboard
- **How:** Just create a Multi-Branch Pipeline - Jenkins handles it automatically!

### Type 2: Parallel Stages (In Jenkinsfile)
- **What:** Multiple stages running simultaneously within one build
- **Where:** Stage View of a single build
- **How:** Use `parallel { }` block in Jenkinsfile

---

## ⚡ Fast Setup Steps

### 1️⃣ Add Generic Jenkinsfile to Both Branches (2 minutes)

**CRITICAL:** Use the SAME generic Jenkinsfile in both branches!

```bash
# Navigate to your repository
cd C:\Users\91817\Devops\Jenkins-parallel-pipeline

# --- Add to feature-login ---
git checkout feature-login
# Copy the generic Jenkinsfile here (same file for both branches!)
git add Jenkinsfile
git commit -m "Add generic Jenkinsfile for multi-branch pipeline"
git push origin feature-login

# --- Add to feature-payment ---
git checkout feature-payment
# Use the SAME generic Jenkinsfile!
git add Jenkinsfile
git commit -m "Add generic Jenkinsfile for multi-branch pipeline"
git push origin feature-payment
```

**Important:** The Jenkinsfile uses `${env.BRANCH_NAME}` - it adapts to whichever branch it runs on!

---

### 2️⃣ Create Multi-Branch Pipeline in Jenkins (2 minutes)

1. Jenkins Dashboard → **New Item**
2. Name: `payment-gateway-app`
3. Type: **Multi-branch Pipeline** → **OK**
4. **Branch Sources** → **Add source** → **Git**
   - Repository URL: `https://github.com/YOUR_USERNAME/Jenkins-parallel-pipeline.git`
   - Or local: `file:///C:/Users/91817/Devops/Jenkins-parallel-pipeline`
5. **Build Configuration**: Script Path = `Jenkinsfile` (default)
6. **Save**

---

### 3️⃣ Scan and Verify (1 minute)

1. On pipeline page → **Scan Multibranch Pipeline Now**
2. Wait 30-60 seconds for scan to complete
3. You should see:
   - `feature-login`
   - `feature-payment`
4. **Both builds start automatically!** ← This is Type 1 Parallelism!

---

## 📊 What You Should See

### Dashboard View (Type 1 Parallelism):
```
payment-gateway-app
  ├── feature-login
  │   └── #1 [Running...] ← Running NOW
  └── feature-payment
      └── #1 [Running...] ← Running NOW (at the same time!)
```

**Both builds running concurrently!** 🎉

### Stage View (Type 2 Parallelism):

Click on `feature-login #1` → **Stage View**:

```
[1. Build] (5 seconds)
    ↓
[2. Test (in Parallel)] (12 seconds - both run together!)
    ├── Unit Tests ──────────┐
    └── Integration Tests ────┼──→ [3. Deploy] (3 seconds)
```

**Parallel stages running side-by-side!** 🎉

### Console Output Examples:

**feature-login build:**
```
BUILDING: App 'payment-gateway' for branch feature-login
Running Unit Tests for feature-login...
Running Integration Tests for feature-login...
DEPLOYING: Branch feature-login to staging.
```

**feature-payment build:**
```
BUILDING: App 'payment-gateway' for branch feature-payment
Running Unit Tests for feature-payment...
Running Integration Tests for feature-payment...
DEPLOYING: Branch feature-payment to staging.
```

**Notice:** Each build only processes its own branch!

---

## ✅ Success Verification

### Type 1 Parallelism (Concurrent Branch Builds):
- [ ] Both `feature-login` and `feature-payment` appear on dashboard
- [ ] Both builds start within seconds of each other
- [ ] Both builds run independently

### Type 2 Parallelism (Parallel Stages):
- [ ] Stage View shows Unit Tests and Integration Tests side-by-side
- [ ] Total test time ≈ 12 seconds (not 22 seconds)
- [ ] Console shows both test stages starting at same time

---

## 🔧 Common Issues & Quick Fixes

| Issue | Quick Fix |
|-------|-----------|
| Branches not found | Click "Scan Multibranch Pipeline Now" |
| Only one branch discovered | Ensure Jenkinsfile exists in both branches |
| Builds not parallel | Check Jenkins executors (Manage Jenkins → Manage Nodes) |
| Wrong branch in logs | Verify using `${env.BRANCH_NAME}` (not hardcoded) |
| Jenkinsfile not found | Check Script Path = `Jenkinsfile` and file exists in repo |

---

## 🎓 Key Concepts to Remember

1. **ONE Generic Jenkinsfile:** Same file in all branches, uses `${env.BRANCH_NAME}`
2. **Type 1 Parallelism:** Automatic - Jenkins builds branches concurrently
3. **Type 2 Parallelism:** Your Jenkinsfile - use `parallel { }` for parallel stages
4. **Each branch is independent:** Jenkins checks out only that branch's code

---

## 📚 Full Documentation
See `JENKINS_SETUP_GUIDE.md` for detailed instructions, explanations, and troubleshooting.

---

## 🎉 You're Done!

After setup, you should see:
- ✅ Concurrent builds on dashboard (Type 1)
- ✅ Parallel stages in Stage View (Type 2)
- ✅ Both branches building automatically on commits
