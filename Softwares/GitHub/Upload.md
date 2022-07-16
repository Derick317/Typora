# Pushing Files to a Repository on GitHub

There are two ways to push files to GitHub, through http, or through ssh.

## SSH

At first, check ssh:

```bash
$ ssh -T git@github.com
```

Usually, you will receive a message as follow:

```bash
git@github.com: Permission denied (publickey).
```

So, you are supposed to type the following commands (and the responses are also shown):

```bash
$ ssh-agent bash
$ ssh-agent -s
SSH_AUTH_SOCK=/tmp/ssh-2ohanOfnx9fL/agent.537175; export SSH_AUTH_SOCK;
SSH_AGENT_PID=537176; export SSH_AGENT_PID;
echo Agent pid 537176;
$ ssh-add ~/.ssh/github/id_rsa # the directory of your 'id_rsa' file
Identity added: ~/.ssh/github/id_rsa (xxx@xxx.com)
```

Then you can check the ssh again:

```bash
$ ssh -T git@github.com
Hi Derick317! You've successfully authenticated, but GitHub does not provide shell access.
```

Now, you can stage the file for commit to your local repository, and commit the file that you've staged in your local repository:

```bash
$ git add -A
$ git commit -m "Add a function in BaseEnv, which is used to change self.step_in_ep"
[main 604a328] Add a function in BaseEnv, which is used to change self.step_in_ep
 1 file changed, 3 insertions(+)
```

Finally, push the changes in your local repository to GitHub.com:

```bash
$ git push -u origin main
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 72 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 493 bytes | 493.00 KiB/s, done.
Total 5 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4), completed with 4 local objects.
To github.com:Derick317/ManiSkill.git
   6f42b7d..604a328  main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

**Note:**

If you type `ssh-add ~/.ssh/id_rsa` before `ssh-agent bash`, you may get `Could not open a connection to your authentication agent.` you just need to type `ssh-agent bash` at first.

