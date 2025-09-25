# Step 0: One-time setup

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git config --global init.defaultBranch main
```

---

# Step 1: Init repo

```bash
git init
```

---

# Step 2: Add file(s)

```bash
echo "# Project" >> README.md
git add .
```


---

# Step 3: Commit

```bash
git commit -m "first commit"
```

---

# Step 4: Branch

```bash
git branch -M main
```

---

# Step 5: Remote

```bash
git remote add origin https://github.com/USERNAME/REPO-NAME.git
```

---

# Step 6: Push

```bash
git push -u origin main
```
