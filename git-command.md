# Git Command Guide

This guide provides a summary of commonly used Git commands, categorized for quick reference and supplemented with brief descriptions of their functionality.

## Common Usage Commands

- **Stage Changes:**
  ```bash
  git add .
  git add <filename>
  ```
  Stages all changes or specific file(s) for the next commit.

- **Commit Changes:**
  ```bash
  git commit -m '<message here>'
  ```
  Saves the staged changes with a descriptive commit message.

- **Push Changes to Remote Repository:**
  ```bash
  git push <remote-name|origin> <branch-name|master>
  ```
  Uploads local branch commits to the remote repository.

- **Pull Changes from Remote Repository:**
  ```bash
  git pull <remote-name|origin> <branch-name|master>
  ```
  Fetches and integrates changes from the specified remote repository branch.

- **Check Repository Status:**
  ```bash
  git status
  ```
  Displays the state of the working directory and staging area, highlighting changes.

- **View Commit History:**
  ```bash
  git log
  ```
  Lists the commit history of the repository.

## Remote Commands

- **Clone a Repository:**
  ```bash
  git clone <url>
  ```
  Creates a copy of a repository from a remote location onto the local machine.

- **Add a New Remote:**
  ```bash
  git remote add <name> <url>
  ```
  Connects a local repository to a remote repository, using the specified URL.

## Branch Management

- **List Branches:**
  ```bash
  git branch
  ```
  Displays all local branches in the repository.

- **Create a New Branch:**
  ```bash
  git branch <branch-name>
  ```
  Creates a new branch with the specified name.

- **Create a Branch from Remote:**
  ```bash
  git branch <branch-name> <remote-name|origin>/<remote-branch-name>
  ```
  Creates a local branch from a specific remote branch.

- **Switch and Create New Branch Simultaneously:**
  ```bash
  git switch -c <branch-name>
  ```
  Creates and switches to a new branch in one command.

- **Rename a Branch:**
  ```bash
  git branch -m <branch-name>
  ```
  Renames the current branch to the specified name.

- **Merge Branches:**
  ```bash
  git merge <branch-name>
  ```
  Combines the changes from the specified branch into the current branch.

## Undoing Changes

- **Unstage Changes:**
  ```bash
  git reset <file>
  ```
  Removes the specified file from the staging area without modifying the working directory.

- **Restore a File:**
  ```bash
  git restore <file-name>
  ```
  Discards changes in a file, restoring it from the last commit.

- **Revert a Commit:**
  ```bash
  git revert <commit>
  ```
  Creates a new commit that undoes all changes made by the specified commit.

## Configuration Commands

- **Set Global Username:**
  ```bash
  git config --global user.name "<name>"
  ```
  Sets the author name to be used for all commits by the current user.

- **Set Global Email:**
  ```bash
  git config --global user.email "<email>"
  ```
  Sets the author email to be used for all commits by the current user.

- **Edit Global Configuration:**
  ```bash
  git config --global --edit
  ```
  Opens the global configuration file in a text editor for manual editing.

- **Set Alias for Commands:**
  ```bash
  git config --global alias.<alias-name> "<git-command>"
  ```
  Creates a shortcut for a Git command. For example, setting `alias.glog` to `log --graph --oneline` makes `git glog` equivalent to `git log --graph --oneline`.

- **Set Default Text Editor:**
  ```bash
  git config --system core.editor <editor>
  ```
  Defines the text editor used for Git commands that require input (e.g., commit messages).

## Git Log Options

- **View Limited Commit History:**
  ```bash
  git log -<limit>
  ```
  Limits the number of commits displayed. For example, `git log -5` shows the last 5 commits.

- **Show Changes in Commits:**
  ```bash
  git log -p
  ```
  Displays the diff introduced in each commit.

- **View Commits by Author:**
  ```bash
  git log --author="<pattern>"
  ```
  Filters commits by a specific author.

- **Search Commits by Message:**
  ```bash
  git log --grep="<pattern>"
  ```
  Displays commits with messages matching the given pattern.

- **View Changes by File:**
  ```bash
  git log -- <file>
  ```
  Shows commits that affected the specified file.

- **Visualize Commit History:**
  ```bash
  git log --graph --decorate
  ```
  Includes a text-based graph and branch/tag decorations in the commit history.

## Git Diff Commands

- **Show Unstaged Changes:**
  ```bash
  git diff
  ```
  Displays the differences between the working directory and the staging area.

- **Show Staged Changes:**
  ```bash
  git diff --cached
  ```
  Displays the differences between the staging area and the last commit.

- **Compare Working Directory to Last Commit:**
  ```bash
  git diff HEAD
  ```
  Shows the changes in the working directory compared to the last commit.

- **Compare Two Branches:**
  ```bash
  git diff <branch1> <branch2>
  ```
  Displays the differences between two branches.

## Rewriting Git History

- **Amend the Last Commit:**
  ```bash
  git commit --amend
  ```
  Updates the last commit with new changes or a revised message.

- **Rebase a Branch:**
  ```bash
  git rebase <base>
  ```
  Moves the branch to start from the specified base commit, applying commits sequentially.

- **Interactive Rebase:**
  ```bash
  git rebase -i <base>
  ```
  Launches an editor to modify commit history interactively.

---

Created by : **Judah Dasuki**