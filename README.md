# My workflow

## Introduction

I was recently asked about my development workflow and here I will try and explain it.

The way I code is very heavily integrated with `git`, I do not use git just as source control.  I view commits as an integral part of the codebase, providing another set of dimension through which the intention and reasoning behind stuff in the codebase may be documentated and viewed.

This is both very important to myself as an author, and to PR reviewers.

I consider being shown how to do this as the single greatest improvement to my work since I first started to code. And a single developer was almost entirely responsible for forcing this into my head.

I also use this flow to some extent for non-coding stuff like admin.

## Tools

There are a few tools I use to make my flow easier, they are listed here:

- A terminal, I prefer the more light-weight ones, and I really dislike those embedded within text editors.
- Git (ofc), I use a local instance even when the organization is stuck using something else for source control.
    - I configure it slighlty to `autosquash` (useful for fixup commits), and a nicer single-line log (`lol`): [gitconfig](https://github.com/AndrewSisley/Home/blob/master/.gitconfig)
- Vim, for git stuff like interactive rebases and commits so that I never have to leave the terminal when doing git stuff. I don't configure anything fancy here, but you can see what I use [here](https://github.com/AndrewSisley/Home/blob/master/.vimrc)
- A text editor (currently VSC), preferably light-weight with no fancy stuff cluttering it or slowing it down (I disable source control and other default stuff in VSC, and I dont use intellisense for some langs like Rust where documentation is well-done-by-default)
- A set of terminal aliases that I use to save typing out common commands, and for reference if I forget anything.  I've built this file up slowly as and when I need, this way I should in theory have a basic understanding of what every command does (instead of just using someone elses file). Two important commands are `e`- to view the alias file, and `ee` to execute it and update the active aliases.  File is run automatically on terminal load.  Mine is [here](https://github.com/AndrewSisley/Home/blob/master/.bash_aliases).

## Rules

I think the best way to explain the workflow itself is to define the set of internal rules I think I try and follow. I've broken these up into two sections.

### End state

These rules describe the end state of the branch, it is what it should look like immediately prior to the opening of a PR.

1. There should be no mistakes.
    1. Every commit should pass the build.
    2. Nothing should be redone, as to redo something implies a mistake was made in an earlier commit.
    3. As well as de-cluttering the branch, this also makes rebases much, much easier as there will be fewer repeat conflicts.
2. Each commit should have sufficient documentation that describes what it is doing and why it does it.
3. It should be entirely possible to split the PR at any given commit.
    1. Each commit should be tested as if only that commit (and the ones before it) are going to be merged.  This is an important component of (1) and (2), as well as (3).
4. Commits should do one useful thing.
    1. This thing should be useful to merge on its own.
    2. It should be useful enough that it is easy to see *why* it is useful by reading the commit message and the PR description alone - viewing the other commits in the PR should not be required in order to see why it is useful.
    3. Refactoring (including formatting), the testing of existing functionality, and introducing new features are different, useful, things.  They should never be in the same commit.
5. Commits should be ordered roughly in increasing relevance to the goal of the PR.
    1. Opportunistic bug fixes that are not related to the goal of the PR come first. This means that if the PR gets split, they will still reach the main branch. They will also not clutter the main body of work.
    2. New tests for existing behaviour should always come before refactoring or new features.  This means that they are provably testing existing behaviour, and not new behaviour.  It also means that if the PR gets split, they will still reach the main branch.
    3. The primary goal of the PR is the last commit.  Everything leading up to it is either opportunistic quick wins, or to support the last commit.  Anything  that comes after this commit is out of scope and should be removed from the branch.

### Work in progress (WIP)

I consider it impossible for anyone to consistently produce the above desired end state in the order in which they code.  In order to produce the desired end state extensive manipulation of the git history is usually required.

I regularly reorder my commits, and I edit them frequently.  I try and track which are PR-ready by flagging thosee that are not with a `WIP - ` prefix - I try and keep the number of these WIP commits small, but often I will have 5 or 6 at one time.  There should be zero by the time the PR is opened.

Aliases and vim speed up this process massively - for example instead of continuously typing out `git commit --amend` I just type `g.ca`, vim ensures I never have to leave the terminal whilst doing so.

The following rules are the ones I think I try and stick to whilst actively working on a task in the hope of reaching the desired end state.

1. Continuously test your work.
    1. The sooner you spot that you have broken something, the cheaper it usually is to fix it.
    2. This should be done in a cost effective, non-distruptive way - I usually just run `go.t` (`go test ./...`, no race flag, no linter) **before** commiting anything.
    3. This can and should be done whilst rebasing too, not just 'normal' coding. You can pause an interactive rebase by inserting `b` into a new line in the rebase file, you can then run the tests to make sure you've not broken anything and then continue (`g.rbc`).
    4. I find integration tests an effective way to test user experience - they should allow you to see what it looks like to work with any interfaces you are modifying and I see this as one of the strongest benefits of a TDD-like set of integration tests, and I will often write a couple to test the UX aspect before bothering to implement the interface.
    5. No change is too small to test, not even a simple rename.
    6. Whilst I do typically write some tests before implementing, I try and keep the number of failing tests as small as possible (i.e. 1 or 0) - add 10 failing tests and it gets tricky to spot when you have broken something that was working.
    7. Try and have the discipline to review the tests for any code area that you plan on changing, before you do the (much more interesting) change itself. If they are insufficient, make them so - it will usually save a lot of time and money vs introducing a regression, and given you more confidence when refactoring.
2. Commit as soon as sensible, for me this is often every couple of minutes or less.
    1. The commit doesn't have to be complete, just the low-level thought (e.g. a for loop, or a rename).
    2. If working on a new approach that you are not sure about instead of immediately editing the target commit, finish the full rework of that commit in a different commit - you can then fixup the target or safely discard the rework once you are certain that you want the rework.
3. Never use `git add .`, instead use `git add -N` (`g.an .`) and `git add -p` (`g.ap`) instead to review what you are staging before you commit it.
4. Push as frequently as possible.
    1. Work done is pretty useless if it is only on your machine, as hopefully only you have access.
    2. Work that only exists on your machine may disappear if your machine does.
    3. Pushing is really really cheap with aliases (`g.p` for a force with lease push) and ssh-agent.
    4. Given (4.3) there is almost never a reason to not push immediately after creating or modifying a commit.
6. Commits should do one useful thing.
    1. I find it much harder to reach this aspect of the desired end-state if this rule is not followed through-out the development process.
    2. I do sometimes get lazy, or have to rapidly context switch, in which case a whole bunch of stuff may temporarily get commited in a single untitled `WIP` commit at the end of the history.  By keeping it at the end of the history it is easy to un-commit with `g.r HEAD~1`, and then re-commit it properly so long as you dont let it grow too much (note: I don't like aliasing away the HEAD~1 for safety, dont want to make it *too* easy to undo commits).
6. Any commit not PR-ready must be prefixed with `WIP - `.
    1. This includes commit level documentation.
    2. This makes it very easy to track what is done, and what is not - it should be difficult to forget to do stuff.
    3. Review any `WIP` commit in Github before removing its `WIP - ` prefix.
7. Prioritize the ordering of commits.
    1. Reordering late can be really expensive, if you know a commit should be before another, move it there as soon as you can regardless of how finished it might be.
    2. This includes fixup commits - I'll usually rebase (`g.rbid`) immediately after creating a fixup commit.  This also helps test as to whether I accidentally included extra stuff based on newer commits in the fixup commit.
    3. It is often a time-saver to re-test as you rebase.
    4. Fixup commits can also be amended (`g.ca`), if you have a bunch of fixes to make to an earlier commit you can create the fixup early (as per (2)) and amend it a few times before rebasing.
8. Make it work, then make it pretty.
    1. This helps reduce the number of failing tests you may have as fast as possible, which makes it easy to tell whenever you break something.
    2. It allows you to move on to more interesting things - for example you may *think* that a refactor will help you implement a feature faster and cleaner, so rush the refactor in a WIP commit, and then rush the implementation of the feature - at which point you will *know* whether the refactor was actually beneficial.
    3. I find it much easier to cleanup something once all the required tests have been written and are passing - you should know very quickly if you break anything during the cleanup due to (1). If your tests are good it often doesnt even require much thought, just delete stuff until the tests start to fail :)
9. Prioritize the cleanliness of the early history.
    1. Building stuff on `WIP` stuff is expensive, so actively try and finish stuff before adding more stuff on top of it.
    2. This includes documentation, as it is important to make sure that the author understands why they did something, not just reviewers.
10. Use `git reset --hard` (`g.r --hard`) to undo anything notable, not Ctrl+z. This is dependent on (2), and I'd strongly recommend running `git status` (`g.s`) and/or `g.an . && g.ap` to scan through your uncommited changes before resetting just in case you have forgotten to commit anything useful :)
