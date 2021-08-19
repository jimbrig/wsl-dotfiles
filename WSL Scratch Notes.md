# WSL Scratch Notes

## Contents

## Commands

### Remove

Use `rm -r` to remove directories:

```bash
# remove directory 'dir'
rm -r dir

# remove file 'test'
rm test

# remove empty directories in dir
rm -d dir
```

## Link

Use `ln` to make hardlinks and symlinks:

```bash
# create symlink from windows ~/Dev dir to WSL ~/dev/windev dir
ln -s /mnt/c/users/jimmy/dev ~/dev/windev

# need sudo for default hard links
sudo ln /etc/wsl.conf ~/.dotfiles/etc/wsl.conf
```

### Tar and Unzip



## DOS vs. UNIX Line  Ending Issues

Unfortunately, the programmers of different operating systems have represented line endings using different sequences:

- All versions of Microsoft Windows represent line endings as CR followed by LF.
- UNIX and UNIX-like operating systems (including Mac OS X) represent line endings as LF alone.

Therefore, a text file prepared in a Windows environment will, when copied to a UNIX-like environment such as WSL, have an unnecessary carriage return character at the end of each line. To make matters worse, this character will normally be invisible, though in some text editors it will show up as ^M or similar.

### Symptoms

If you run a script initially created on windows with `` line ending support, you will see errors like this in WSL:

- *from `install-homebrew.sh`:*

```bash
ERROR:
./install-homebrew-ubuntu.sh: line 4: $'\r':

ERROR:
./install-homebrew-ubuntu.sh: line 7: $'\r': command not found
./install-homebrew-ubuntu.sh: line 15: $'\r': command not found
./install-homebrew-ubuntu.sh: line 10: $'\r': command not found
./install-homebrew-ubuntu.sh: line 12: $'\r': command not found
./install-homebrew-ubuntu.sh: line 15: $'\r': command not found
./install-homebrew-ubuntu.sh: line 20: $'\r': command not found
/bin/bash: -c: line 856: syntax error: unexpected end of file

./install-homebrew-ubuntu.sh: line 17: $'\r': command not found. Did you mean gcc, gcc@9, gcc@8, gcc@7, gcc@6 or gcc@5?
```

- *from `install-gh-cli.sh`*:

```bash
./install-github-cli.sh: line 2: $'\r': command not found
1.14.0
./install-github-cli.sh: line 4: $'\r': command not found
curl: (3) URL using bad/illegal format or missing URL
./install-github-cli.sh: line 6: $'\r': command not found
tar: gh_1.14.0\r_linux_amd64.tar.gz\r: Cannot open: No such file or directory
tar: Error is not recoverable: exiting now
./install-github-cli.sh: line 8: $'\r': command not found
cp: cannot stat 'gh_1.14.0'$'\r''_linux_amd64/bin/gh': No such file or directory
./install-github-cli.sh: line 10: $'\r': command not found
./install-github-cli.sh: line 11: gh: command not found
./install-github-cli.sh: line 12: $'\r': command not found
cp: cannot stat 'gh_1.14.0'$'\r''_linux_amd64/share/man/man1/*': No such file or directory
./install-github-cli.sh: line 14: $'\r': command not found
```

### Solution: `dos2unix`

>  Source: [How do I fix "$'\r': command not found" errors running Bash scripts in WSL? - Ask Ubuntu](https://askubuntu.com/questions/966488/how-do-i-fix-r-command-not-found-errors-running-bash-scripts-in-wsl#:~:text=steeldriver is correct that the problem is that,is absent in traditional Unix-style line endings (LF).)

`$'\r': command not found` strongly suggests the issue is that you have used a Windows text editor that has saved your files with DOS-style CRLF line endings - see for example [DOS vs. Unix Line Endings](http://www.cs.toronto.edu/~krueger/csc209h/tut/line-endings.html)

Inside WSL:

```bash
sudo apt-get install dos2unix
```

Then,

```bash
dos2unix [file]
```

Full documentation:

```bash
man dos2unix
```

Here you can see it in action:

![image-20210819020049489](C:\Users\jimmy\AppData\Roaming\Typora\typora-user-images\image-20210819020049489.png)

now try to re-run scripts:

```bash

```



## Permissions

After linking my ssh keys between windows and WSL via: `ln /mnt/c/users/jimmy/.ssh ~/.ssh` I received the error:

```bash
Warning: Permanently added the RSA host key for IP address '140.82.113.3' to the list of known hosts.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0777 for '/home/jimbrig/.ssh/id_ed25519' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/home/jimbrig/.ssh/id_ed25519": bad permissions
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

Need to change from a symlink to hardlink and/or jsut copy via `cp`:

```bash
rm -r ~/.ssh
cp -r /mnt/c/users/jimmy/.ssh ~/.ssh
```

Then fix permissions via:

```bash
chmod 600 ~/.ssh/id_rsa
```

> What this does is set Read/Write access for the owner, and no access for anyone else. That means that nobody but you can see this key. The way god intended.
>
> *Source  [Sharing SSH keys between Windows and WSL 2 | Windows Command Line (microsoft.com)](https://devblogs.microsoft.com/commandline/sharing-ssh-keys-between-windows-and-wsl-2/)*

Now it works: ✔️

![image-20210819023041641](C:\Users\jimmy\AppData\Roaming\Typora\typora-user-images\image-20210819023041641.png)

Fix GPG keys also:



## %PATH%

Warning: /home/linuxbrew/.linuxbrew/bin is not in your PATH.

```bash
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/jimbrig/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

