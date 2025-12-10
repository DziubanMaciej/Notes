parentTopic:: [[Computer]]

# Bash prompt on Windows Git Bash

Insert this to your .profile
```bash
construct_bash_prompt() {
    PS1="\[\033]0;$TITLEPREFIX:$PWD\007\]\[\033[32m\]\u@\h \[\033[35m\]\[\033[33m\]\w\[\033[36m\]`__git_ps1`\[\033[0m\] \$ "
}
PROMPT_COMMAND="construct_bash_prompt"
```