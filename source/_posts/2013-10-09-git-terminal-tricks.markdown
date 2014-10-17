---
layout: post
title: "Git Terminal Tricks"
date: 2013-10-09 10:12
comments: true
categories: Tip
---
I develop on Mac OS X and do my Git stuff in Terminal but I often found myself cursing at two things:

- Doing `git status` or `git branch` just to know which branch was active
- Doing `git checkout d`<kbd>Tab</kbd> in the hopes of getting `git checkout develop`

This might be horribly obvious to some people and don't ask me why, but only after months of cursing at this, I decided to Google these two things and make my Git life easier. 
<!-- more -->
## Git auto-complete
For the auto-complete, I found this stackexchange question: [git auto-complete for *branches* at the command line?](http://apple.stackexchange.com/questions/55875/git-auto-complete-for-branches-at-the-command-line). [Michael Durrant](http://apple.stackexchange.com/users/24565/michael-durrant)'s answer is surprisingly simple, but works just fine: download a git-autocomplete bash script and call it in your `~/.bash_profile` file. 

    curl https://raw.github.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash
    
Edit (or create) your `~/.bash_profile` file with your favorite editor (Vim, Nano or [maybe even Sublime?](http://www.sublimetext.com/docs/2/osx_command_line.html)) and paste the following:

    if [ -f ~/.git-completion.bash ]; then
        . ~/.git-completion.bash
    fi
> Note that this not only works for branches. If you do `git  `<kbd>Tab</kbd><kbd>Tab</kbd> (note the space after `git`), you get a list of all git sub-commands. `git bl`<kbd>Tab</kbd> would auto-complete to `git blame`. 

## Displaying branch names in Terminal
To display branch names in terminal (next to the current directory), I used a solution found [here](http://martinfitzpatrick.name/article/add-git-branch-name-to-terminal-prompt-mac). Simply copy/paste the following lines in the same `~/.bash_profile` as before

   	parse_git_branch() {
	    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
	}
	export PS1="\u@\h \W\[\033[32m\]\$(parse_git_branch)\[\033[00m\] $ "

> Notes
> 
> - If you already have a PS1 variable, just add the `\$(parse_git_branch)` somewhere in there.  
> - `\[\033[32m\]` sets the color of the branch name to green. `\[\033[00m\]` sets everything back to black. To change the color of the branch name, change the `2` in the `[32m\]` bit. You can find the [color codes on Wikipedia](http://en.wikipedia.org/wiki/ANSI_escape_code#Colors).
> - You do not need to restart Terminal or anything for the changes to take effect, just do `source ~/.bash_profile`.