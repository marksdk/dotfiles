# Setting up a new machine

Run this command:

```zsh
curl -Lks https://raw.githubusercontent.com/marksdk/dotfiles/master/install.sh | /bin/bash -x
```

This is what it does:

1. Install Homebrew
2. Add `mas` to Homebrew so it can install apps from the Mac App Store programatically
3. Install `taps`, `brews`, and `casks` from `Brewfile`
4. Install `oh-my-zsh`
5. Install `Pure` prompt
6. Source `dotfiles` and do the following:
   1. Add `dot` alias to `.zshrc`:
      - `echo "alias dot='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'" >> .zshrc`
   2. Ignore `.dotfiles` to avoid recursion issues:
      - `echo .dotfiles >> .gitignore`
   3. Clone dotfiles
      - `git clone --bare git@github.com:marksdk/dotfiles.git $HOME/.dotfiles`
   4. Define the alias in the current shell:
      - `alias dot='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'`
   5. Checkout the content from the bare repo to $HOME:
      - `dot checkout`
   6. This might cause an error because there are already conflicting dotfiles. One solution:
      - `mkdir -p .dotfiles-backup && \ dot checkout 2>&1 | egrep "\s+\." | awk {'print $1'} | \ xargs -I{} mv {} .dotfiles-backup/{}`
   7. Re-run the check if there are problems:
      - `dot checkout`
   8. Set the flag showUntrackedFiles to no on this specific (local) repository:
      - `dot config --local status.showUntrackedFiles no`
7. Install Visual Studio Code extension to sync all settings and extensions:
   - `code --install-extension shan.code-settings-sync`

## install.sh

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install mas
brew bundle
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
source ~/.zshrc
mkdir -p "$HOME/.zsh" && git clone https://github.com/sindresorhus/pure.git "$HOME/.zsh/pure"
echo "\nfpath+=$HOME/.zsh/pure \nautoload -U promptinit; promptinit \nprompt pure" >> $HOME/.zshrc
git clone --bare https://github.com/marksdk/dotfiles.git $HOME/.dotfiles
function dot {
   /usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME $@
}
mkdir -p .dotfiles-backup
dot checkout
if [ $? = 0 ]; then
  echo "Checked out dotfiles.";
  else
    echo "Backing up pre-existing dotfiles.";
    dot checkout 2>&1 | egrep "\s+\." | awk {'print $1'} | xargs -I{} mv {} .dotfiles-backup/{}
fi;
dot checkout
dot config status.showUntrackedFiles no
```

## Sources

The idea of using the `$HOME` dir as a sort-of-but-not-really git repository comes from this article: <https://www.atlassian.com/git/tutorials/dotfiles>. It removes the need for symlinking which was messed up for me, for some—still—unknown reason. This article is based on this comment on Hacker News: <https://news.ycombinator.com/item?id=11070797> and I read about it here first: <https://www.anand-iyer.com/blog/2018/a-simpler-way-to-manage-your-dotfiles.html>.

Throughout, my script refers to the directory `.dotfiles`, and the `git` command that works on that directory is aliased with `dot`. The articles refer to `.cfg` and `config`.
