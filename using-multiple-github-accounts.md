## Problem
You have two different Github accounts that you want to use on the same computer with SSH keys.
Perhaps one is for personal projects and one is for work.

## Solution Summary
You can use two different SSH keys, and modify your SSH & Git configs to allow you to use different accounts
depending on where on your computer the repository lives.

## Solution Details

1. [Create an SSH key pair for each account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent),
    one for each account, and then [add each of them to their account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).
    
    You can go to the linked articles for information on how to do each of these.
    You should save each key with a different name

2. Edit your ssh config file (`~/.ssh/config` on most Unix systems — such as Linux, BSD, and macOS)
```
# For your first account, (for me, my personal account)
Host github.com
  IdentityFile ~/.ssh/id_github-personal
  IdentitiesOnly yes # This prevents git from searching for other keys to use in your ~/.ssh folder
  
# For your second account, (for me, my work account)
Host github-work
  HostName github.com # this tells ssh to use this hostname instead of the alias.
  IdentitiesFile ~/.ssh/id_github-work
  IdentitiesOnly yes
```

3. If you use an SSH agent, you can add the keys to it.

This is only necessary if you use a passphrase on your keys (you probably should!)
```
$ ssh-add ~/.ssh/id_github-personal ~/.ssh/id_github-work
```

4. Test that your SSH config has worked
```
$ ssh -T git@github.com
$ ssh -T git@github-work
```

You should see a message like the following:
```
Hi JaseHadd! You've successfully authenticated, but GitHub does not provide shell access.
```

Each command should show a different username in the response from GitHub.

5. Git Config (optional)
This step is optional — if you want, it should already work if you use the alias you made in the
SSH config, and tested in the previous step. To do this, replace `github.com` with `github-work` in
your remotes.

Edit your `.gitconfig` in your home directory. You'll be changing the `[user]` section to match your
_personal_ details, and adding an `[IncludeIf]` section – change the `~/WorkStuff/` to whatever
directory you keep your work projects in.
```
[user]
  name = Your Name
  email = your.personal@ema.il
[IncludeIf "gitdir:~/WorkStuff/"]
  path = ~/.gitconfig-work
```

We've referenced a file called `.gitconfig-work` in the config above, so let's make it in your
home directory, with contents like the following:
```
[user]
  name = Your Name
  email = your.work@ema.il
[url "git@github-work:"]
  insteadOf = git@github.com
```

With this step done, you can use the regular remote URLs and Git & SSH will swap it all
around for you in the background.

Let me know if there are any issues.

## Acknowledgements
I drew inspiration and the main structure of the solution from
[@oanhnn](https://gist.github.com/oanhnn/80a89405ab9023894df7).

The steps are largely the same, changed a bit to better suit how I work and with the URL swapping added.
