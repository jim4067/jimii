## Adding a GPG KEY to GitHub

The first step is to check whether you have a gpg key that you want to use, in which case you can skip to step two.

Using

```bash
gpg --list-secret-keys --keyid-format=long
```

will list all the gpg keys on your system

### 1. Creating GPG Keys

To create a new GPG key open your terminal and run (for versions 2.1.17 or higher). you can check your version using `gpg--version`

```bash
gpg --full-generate-key
```

or (for versions lower than 2.1.17)

```bash
gpg --default-new-key-algo rsa4096 --gen-key
```

This will prompt you to select the GPG algorithm. The default is RSA(It has much criticism but we will use it).

Unless you are aware of what you are doing the defaults will work just fine.

Press Enter for every prompt, Enter your name, email adress and comment for what you will be using the key for.

You should have something similar to this
!!!!IMAGE HERE

Confirm that your details are okay and press enter.
You will be prompoted for a passprase. Kind of a password. Create one and complete the setup

You should have something like this
!!!IMAGE HERE

### 2. Adding GPG to GitHub

Go to GitHub settings and in the sidebar, under the access section and go to the `SSH and GPG keys` tab

Alternatively 

You could use the command pallet. To access the command pallet for GitHub
`Ctrl + k ` 

To go directly to the GPG settings tab type 
`Settings > SSH and GPG Keys`

Click `New GPG Key`

Go back to your terminal and type 
`gpg --armor --export <Key Id>`
Our Key Id in this case is `2A671563F66799C9CE8E2027609E6CBC08B205A0`

you should have something like this
IMAGE HERE

Copy everything from `-----BEGIN PGP PUBLIC KEY BLOCK-----` to `-----END PGP PUBLIC KEY BLOCK-----`

back to GitHub. Give your GPG key a name and paste the block you just copied to the Key field

Add GPG Key and you now have a gitHub GPG Key.

### 3 configuring git with GPG Key.
I contribute to open source and most projects require you to sign you commits. Here is how I configured git to sign commits with the gpg key.

set the commit.gpgsign var to true
```bash 
git config --global commit.gpgsign true
```
Since I do not want to sign commits on a per repo basis. If you don't want to set this globally you could use
```bash
git config commit.gpgsign true
```

Finally set the sign key using 
```bash
git config --global user.signkey 2A671563F66799C9CE8E2027609E6CBC08B205A0
```

### 4 Errors encountered
I came across this problem while trying to sign my first commit with my GPG Key.

```bash
    error: gpg failed to sign the data
    fatal: failed to write commit object
```

To debug it, check what git is doing by logging it,

```bash
    GIT_TRACE=1 git commit -m "your commit message"
```

```bash
    11:19:24.936571 git.c:455 trace: built-in: git commit -m -bsau
```

For some reason, I had not added my GPG key to git.

To check if this is the case run a

```bash
    git config --global --list
```

which will list all the global vars that are set to git.
