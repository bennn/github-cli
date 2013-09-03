github-cli
==========

My favorite command line interface for working with github.
Wraps ye ordinary git commands to support a nice branching git repository that harnesses the awesome power of github.

# Install
Just stick a copy of the `git-cli` script and `my-repo.ini` file into your repo.
Fill out the fields in the .ini and optionally rename these files to something more appropriate for your project.
Make new labels "hotfix", "pending-review", and "resolved" on github.
Now you can execute the script using the commands detailed below.

# The short story
After you install this script, your workflow ought to be:
* Create an issue on github, assign it to a milestone branch
* `git-cli checkout <issue-number>`
* Hack hack hack. `git add -A`
* `git-cli commit "message"`
* `git-cli review`
* Revise, rehack
* `git-cli close`

The `git-cli` commands are made to handle all the boring branch maintenance and syncing with origin.
You, the user, can just `checkout` the appropriate issue to start working.
Once you are ready to push to origin, run `review` to create a pull request on github.
Finally, `close` merges the issue branch to the milestone branch.

`commit` is a bit nicer than the regular old `git commit` because it includes the issue number at the start of the commit message and echos your message before committing, so you can eyeball it for aesthetic purposes.

# Commands Detail

Strings within `code blocks` denote shell commands, strings within `[braces]` denote optional arguments, and strings within `<angle-braces>` describe mandatory arguments.
The optional arguments may be inserted directly into a command, but the mandatory arguments cannot; instead you have to parse them manually and figure out what to insert.
For example, `mycommand [-o] <filename>` means that when you execute `mycommand`, it needs to be given the name of a file.
`mycommand foo.txt` would work, assuming "foo.txt" exists.
Optionally, you may run `mycommand -o foo.txt`.
(And who knows what'll happen if you try `mycommand --iluvfood foo.txt`.)

## Status
`git-cli status` is exactly the same as git status, except it prints blue text.
Maybe someday `git-cli` by itself will print out the git status too.

## Checkout
### Usage
`git-cli checkout [--no-pull] <issue-number>` prepares your repository so you may begin work on an issue.
The issue number is assigned automatically by github.

By default, this command pulls outstanding refs into the issue branch.
So if you started work on issue "michellebranch-23", switched gears and merged "michellebranch-25" into the parent "michellebranch", then running `git-cli checkout 23` would bring you back to "michellebranch-23" and ALSO pull in the changes you just committed in issue #25.
Use the `--no-pull` option to override this behavior and stick to the local stuff.

Command first determines the milestone associated with the supplied issue number, defaulting to master.
If the inferred milestone is master, the checkout will fail unless the issue is labeled as a "hotfix".
Otherwise, command checks out the appropriate branch, creating it locally if it did not previously exist, and then checks out the issue branch. 

### Replaces
Normally one would use `git checkout <branch-name>` to switch branches and `git checkout -b <branch-name>` to create a new branch and switch to it.
Git does not provide any way of pulling in existing refs when you check out a branch, so you'd have to take care of that manually with a bunch of calls to `git fetch` and `git merge`.

## Add
### Usage
`git-cli add <pattern>` stages the changes to all tracked files beginning with the string `<pattern>` in the current directory tree.

This command supports all the same options as `git add`, but since the only option I ever use is `-A`, as in `git add -A` (to stage ALL my changes for commit), and prefer that, shorter command to `git-cli add -A`, I offer no guidance here.

### Replaces
Normally, one adds files using `git add <filename>`, where filename is the path to and name of the file.
This is pretty good, but the filenames can get long and are often similar.
Case in point: if you've made changes to "./2013fa/hw/ps1/release/mymodule.ml" and "./2013fa/hw/ps1/release/mymodule.mli", you have two options:
```
git add ./2013fa/hw/ps1/release/mymodule.ml
git add ./2013fa/hw/ps1/release/mymodule.mli
```
- or -
```
git-cli add mymodule
```

Well, technically you have 3 options.
You could also `git add -A` and you're cool.
UNLESS you had changes to a bunch of files and only wanted to include those two in your commit, which I very often want to do. 

## Commit
### Usage
`git-cli commit "<commit-message>"` commits all changes staged for commit and prepends the current issue number to the commit message.
Before making the commit for real, it echoes the commit message and demands a "y" at the prompt.

This command also technically supports all the same options as `git commit`, but I haven't tried them yet.

### Replaces
`git commit -m "<commit-message>"` Here the option `-m` is not required; by default, git opens your preferred text editor and asks you to write a snazzy message.
This is fine, but a little annoying.
Also `git commit` does not demand that you reference the issue you are working on in the commit message, which is BAD BAD BAD.

### Writing a good commit message
Commit messages should be about one line long, reference the current issue with a #<issue-number>, and succinctly describe all the changes to files being committed. 

What should you do if you have so many changes that you cannot possibly describe them in 80 character? Abort your commit, unstage some files, and commit a block of work that you CAN describe.
There's no need to commit everything all at once.
Commit files in tiny groups or even one at a time and take care to document your changes.

`git status` describes how to unstage a file for commit, allowing you to add it again later.

## Review
### Usage
`git-cli review`.
That's all.
No arguments, no options.
The command syncs your local issue branch with the origin so everyone can see it, creates a pull request on github complete with a line-by-line diff, and labels the issue as "pending-review".

### Replaces
Ummmm I'm not entirely sure.
Synchronization with origin is done via `git push origin/<issue-branch>`, but to generate a pull request one normally has to work with the github UI, click the right buttons and specify the correct "merge-from" and "merge-to" branches to compare for the diff.
It's a real pain.
Thankfully the api is easy to deal with; you don't have to remember how to anything.

## Close
### Usage
`git-cli close` doesn't take any arguments either, syncs your local branch to the origin, merges the existing pull request with the parent, labels the issue as "resolved", and removes the "pending-review" label.
If you were working on issue "zardoz-42" and had posted a review, `git-cli close` would pull the changes is "zardoz-42" to the branch "zardoz". 

Note that this command does not delete the issue branch "zardoz-42" from your local branch history.
That still needs to be done manually, when you're good and ready, with `git branch -d zardoz-42`.
Or not.
You can let branches pile up and it won't affect things much.

### Replaces
This command replaces a whole slew of checkout and push commands.
`git checkout parent`, `git pull origin/parent`, `git merge origin/child-issue`.
(Nevermind the additional check to make sure that "parent" is synced with "master".)
Plus there's the relabeling of issues.
Just use `git-cli close` and sleep better at night.

# References
* Vincent Driessen's [Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)

# One more thing
Don't forget to add your .ini file to the gitignore. Have fun.
<!-- Ben Greenman 2013-09-02 -->

