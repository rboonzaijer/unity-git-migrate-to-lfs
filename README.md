# Unity - Migrate an existing repository to LFS

### Requirements

- curl
- python3
- (works on WSL2)

# Example walkthrough

### Clone your non-LFS repository to a new folder (the safest way)

```bash
mkdir -p ~/temp-unity-lfs-tests

cd ~/temp-unity-lfs-tests

git clone git@github.com:rboonzaijer/unity-git-migrate-to-lfs-test-original.git

cd unity-git-migrate-to-lfs-test-original
```

### Make sure you pull all branches

- https://stackoverflow.com/questions/10312521/how-do-i-fetch-all-git-branches

```bash
git branch -r | grep -v '\->'  | grep -v `git branch | awk '/\*/ { print $2; }'`| while read remote; do git branch --track "${remote#origin/}" "$remote"; done
```

### Fetch all remotes

```bash
git fetch --all
```

### Pull all remotes
```bash
git pull --all
```

### Create a clean remote repository for the final results

- https://github.com/rboonzaijer/unity-git-migrate-to-lfs-test-final-result-lfs

### Download .gitattributes for Unity

```bash
curl -o ___temp___.gitattributes https://raw.githubusercontent.com/rboonzaijer/unity-quickstart/main/all-unity-root-files/.gitattributes
```

### Download history-rewrite tool

- https://github.com/rboonzaijer/git-filter-repo/ (fork)
- or latest from original repo: `curl -o ___temp___git-filter-repo https://raw.githubusercontent.com/newren/git-filter-repo/main/git-filter-repo`

```bash
# forked (tested/working)
curl -o ___temp___git-filter-repo https://raw.githubusercontent.com/rboonzaijer/git-filter-repo/main/git-filter-repo
```

### Rewrite git commit history: Add the '.gitattributes' to the first commit

- https://github.com/rboonzaijer/git-filter-repo/blob/main/contrib/filter-repo-demos/insert-beginning#L17

```bash
python3 ___temp___git-filter-repo --force --commit-callback "if not commit.parents: commit.file_changes.append(FileChange(b'M', b'.gitattributes', bytes('$(git hash-object -w '___temp___.gitattributes')',encoding='utf-8'), b'100644'))"
```

### Remove temp downloaded files

```bash
rm ___temp___*
```

### LFS migrate all branches (--everything) + the whole history (--fixup)

```bash
git lfs migrate import --everything --fixup -y
```

### Optional: check model.fbx after the migrate
```bash
cat model.fbx

# expect:
# version https://git-lfs.github.com/spec/v1
# oid sha256:4a58b13d1ac7ae9ac715d01d5cba13219bfa0cb3eadea49c3ebcf646b6d7904d
# size 32
```

### Populate working copy with real content from Git LFS files

```bash
git lfs checkout
```

### Check model.fbx after populate

```bash
cat model.fbx

# expect:
# new contents of model.fbx (LFS)
```

### Cleanup

- https://github.com/git-lfs/git-lfs/wiki/Tutorial#cleaning-up-the-git-directory-after-migrating

```bash
git reflog expire --expire-unreachable=now --all && git gc --prune=now
```

### List all lfs files (current branch)

```bash
git lfs ls-files --size

# expect:
# 4a58b13d1a * model.fbx (32 B)
```

### List all lfs files (also other branches)

```bash
git lfs ls-files --size --all

# expect:
# 04f2c7a8b1 - feature-branch-model2.fbx (70 B)
# 4a58b13d1a * model.fbx (32 B)
# 694adb5058 * model.fbx (32 B)
# bf4942b2ec - feature-branch-model2.fbx (58 B)
```

### Push to a new repository

```bash
git remote add origin git@github.com:rboonzaijer/unity-git-migrate-to-lfs-test-final-result-lfs.git
git push -u origin main
git push --all
```

### Check the repository manually

- Initial commit should have the .gitattributes: https://github.com/rboonzaijer/unity-git-migrate-to-lfs-test-final-result-lfs/commit/367c032946705231e13317203495dafba9072e07
- Model should have 'Stored with Git LFS': https://github.com/rboonzaijer/unity-git-migrate-to-lfs-test-final-result-lfs/blob/main/model.fbx
- Model in other branch should have 'Stored with Git LFS': https://github.com/rboonzaijer/unity-git-migrate-to-lfs-test-final-result-lfs/blob/feature/model2/feature-branch-model2.fbx
- README.md should just be stored normally: https://github.com/rboonzaijer/unity-git-migrate-to-lfs-test-final-result-lfs/blob/main/README.md
- optionally, if no LFS files are found, you could try this: `git lfs push origin --all`

### Sanity check - Clone your final repo

```bash
cd ~/temp-unity-lfs-tests
git clone git@github.com:rboonzaijer/unity-git-migrate-to-lfs-test-final-result-lfs.git
cd unity-git-migrate-to-lfs-test-final-result-lfs

cat model.fbx
# should be:
# new contents of model.fbx (LFS)

git checkout feature/model2
cat feature-branch-model2.fbx
# should be:
# new contents of model2.fbx (should be stored on LFS also, bigger file

git checkout main
git lfs ls-files --size
# expect:
# 4a58b13d1a * model.fbx (32 B)

git lfs ls-files --size --all
# expect:
# 04f2c7a8b1 - feature-branch-model2.fbx (70 B)
# 4a58b13d1a * model.fbx (32 B)
# 694adb5058 * model.fbx (32 B)
# bf4942b2ec - feature-branch-model2.fbx (58 B)

echo 'new' > model3.fbx
git add .
git status -u
git commit -m 'new model'
git push
```

### Last check:

- Model3 should have 'Stored with Git LFS': https://github.com/rboonzaijer/unity-git-migrate-to-lfs-test-final-result-lfs/blob/main/model3.fbx

# Done

### Remove all test files for this example

```bash
rm -rf ~/temp-unity-lfs-tests/
```
