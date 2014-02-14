# github-cli

My favorite command line interface for working with github.
Wraps ordinary git commands to support a nice branching, github-centric model.

## Install
Just stick a copy of the `github-cli.exe` script and `my-repo.ini` file into your repo.
First, fill out the fields of the .ini.
You're encouraged to rename them into something more suitable or mnemonic, but be careful that the variable `INI_FILE` at the top of the executable matches the name of the .ini file.
Next, make new labels "hotfix", "pending-review", and "resolved" on github.
Now you can execute the script using the commands detailed below.

## Dependencies

* Python 3.3.4
* The [requests](http://docs.python-requests.org/) library

## The short story
After you install this script, your workflow ought to be:
* Create an issue on github, assign it to a milestone
* Execute `github-cli.exe checkout <issue-number>`
* Hack hack hack. Stage changes with `github-cli.exe add -A` or something similar.
* Commit the changes. `github-cli.exe commit "message"`. The message is required.
* Post a pull request from the issue branch to the milestone branch. `github-cli.exe review`
* Examine the pull request. Have a friend examine the pull request. Make any extra additions or commits that seem appropriate. Repeat.
* Merge the issue branch. `github-cli.exe close`

The `github-cli.exe` commands are made to handle all the branch maintenance and keep synced with origin.
Just `checkout` the appropriate issue to start working.
Once you are ready to push to origin, run `review` to create a pull request on github.
Finally, `close` merges the issue branch to the milestone branch.

`commit` is a bit nicer than the regular old `git commit` because it includes the issue number at the start of the commit message and echos your message before committing.
`add` treats its argument as a prefix, matching all files whose path (from the current directory) matches the argument string.

## Commands Detail

Strings within `code blocks` denote shell commands, strings within `[braces]` denote optional arguments, and strings within `<angle-braces>` describe mandatory arguments.
The optional arguments may be inserted directly into a command, but the mandatory arguments cannot; instead you have to parse them manually and figure out what to insert.
For example, `mycommand [-o] <filename>` means that when you execute `mycommand`, it needs to be given the name of a file.
`mycommand foo.txt` would work, assuming "foo.txt" exists; running `mycommand -o foo.txt` is another choice.

### Status
`github-cli.exe status` is exactly the same as git status, except it prints blue text.
The alias `github-cli.exe st` is also supported.
This command exists to make `github-cli.exe` self-sufficient.

### Refresh
`github-cli.exe refresh` pulls in new changes to your branch and to master.
Use it when another user has committed to the branch you are working on and you want to stay updated.
It is less useful, conceptually, for syncing with origin; `review` will also sync.

### Checkout
#### Usage
`github-cli.exe checkout [--no-pull] <issue-number>` checks out (optionally creating) the issue branch associated with issue number `<issue-number>`.
The issue number is assigned automatically by github.

Say you have checked out issue #9001.
Executing `github-cli.exe commit "fixed everything"` and entering a "y" when prompted would create a commit with message "[#9001] fixed everything".
This way, commits show up under their issues in github.

#### Replaces
`git commit -m "<commit-message>"` Here the option `-m` is not required; by default, git opens your preferred text editor and asks you to write a snazzy message.
This is fine, but a little annoying.
Also `git commit` does not demand that you reference the issue you are working on in the commit message.

#### Writing a good commit message
Commit messages should be about one line long, reference the current issue with a #<issue-number>, and succinctly describe all the changes to files being committed.

What should you do if you have so many changes that you cannot possibly describe them in 80 character? Abort your commit, unstage some files, and commit a block of work that you can describe.
There's no need to commit everything all at once.
Commit files in tiny groups or even one at a time and take care to document your changes.

### Review
#### Usage
`github-cli.exe review`.
That's all.
No arguments, no options.
The command syncs your local issue branch with the origin so everyone can see it, creates a pull request on github complete with a line-by-line diff, and labels the issue as "pending-review".

#### Replaces
There is no simple equivalent.
Synchronization with origin is done via `git push origin/<issue-branch>`, but to generate a pull request one normally has to work with the github UI, click the right buttons and specify the correct "merge-from" and "merge-to" branches to compare for the diff.

This script plugs in to the github api and takes care of all that for you.

### Close
#### Usage
`github-cli.exe close` doesn't take any arguments either, syncs your local branch to the origin, merges the existing pull request with the parent, labels the issue as "resolved", and removes the "pending-review" label.
If you were working on issue "zardoz-42" and had posted a review, `github-cli.exe close` would pull the changes is "zardoz-42" to the branch "zardoz". 

Note that this command does not delete the issue branch "zardoz-42" from your local branch history.
That still needs to be done manually, when you're good and ready, with `git branch -d zardoz-42`.
Or not.
You can let branches pile up and it won't affect things much.

#### Replaces
This command replaces a whole slew of checkout and push commands.
`git checkout parent`, `git pull origin/parent`, `git merge origin/child-issue`.
(Nevermind the additional check to make sure that "parent" is synced with "master".)
Plus there's the relabeling of issues.
Just use `github-cli.exe close` and sleep better at night.

## FAQ

### Q: How do I install the requests library?

There are instructions on the library's [site](http://docs.python-requests.org/), but what's worked for me was to install via pip:
```pip-3.3 install git+https://github.com/kennethreitz/requests```
and then add the python3.3 packages to my `PATH`:
```export PATH=$PATH:/usr/local/lib/python3.3/site-packages```

### Q: I need to commit to master. It's an emergency!

Label the issue you want to commit to as a "hotfix" on github.
The script allows hotfixes to branch off master and merge into it directly.

## References
* Vincent Driessen's [Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)

## One more thing
Don't forget to add your .ini file to the gitignore. Have fun.
<!-- Ben Greenman 2014-02-14 -->

