# Working with a Private Registry and/or Private Repositories

If the registry needs to be private and/or privately hosted packages are
added to it, using the git ssh protocol works well with Julia's Pkg manager:

## 1. Add Packages to the Registry with the git ssh url Protocol

Activate the private registry in Julia with the `git@github.com:...`
url format
```
using Pkg
pkg"registry add git@github.com:User/CustomRegistry.git"
```

Adding a private package to the registry with the `git@github.com:...`
url format
```
using LocalRegistry
register(package, registry, repo="git@github.com:User/Package.jl.git")
```

## 2. Using an External git Binary with Julia's Package Manager

By default Julia's package manager uses libgit2 and libssh2 for git
operations. The remaining items discuss how to set up ssh keys to work
with these libraries. However, for Julia 1.7 and later, a potentially
much simpler and more robust solution is to set the environment
variable
```
JULIA_PKG_USE_CLI_GIT=true
```
to make Pkg use an external git binary.

To reduce the number of headaches associated with this, add the following to your startup.jl file:
```
ENV["JULIA_PKG_USE_CLI_GIT"]=true
```

## 3. Set Up Persistent git ssh Authentication that Julia Recognizes

By default libssh2 (which is used by the Julia Pkg manager via libgit2)
looks for git ssh keys in `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub` so if you
have that typical setup you shouldn't need to do anything extra.

If that doesn't work, or if your git keys need to be named differently,
you can try to set `SSH_PUB_KEY_PATH` and `SSH_KEY_PATH` environmental
variables:

In Juno, setting “Julia Options” > “Arguments” to:
```
SSH_PUB_KEY_PATH=~/.ssh/key.pub, SSH_KEY_PATH=~/.ssh/key
```
or in `~/.bashrc`, for instance
```
export SSH_PUB_KEY_PATH=~/.ssh/key.pub
export SSH_KEY_PATH=~/.ssh/key
```

## 4. Generating a Compatible Key

libssh2 is much less flexible than normal ssh in many ways, including
keys. This is a way to generate a working key:
```
ssh-keygen -t rsa -b 4096 -m PEM
```
Importantly `-m PEM` is needed to store the key in the OpenSSL PEM
format rather than the default (from OpenSSH 7.8) OpenSSH format. This
is an unfortunate tradeoff since the OpenSSH format is more secure,
but libssh2 does not understand it.

If you already have a key, you can recognize the difference from the
first line of the private key:
* OpenSSL PEM format begins with `-----BEGIN RSA PRIVATE KEY-----`
* OpenSSH format begins with `-----BEGIN OPENSSH PRIVATE KEY-----`

## 5. Extracting the Public Key from the Private Key

If you only have the private key available, the corresponding public
key can be generated from the private key with
```
ssh-keygen -y -f ~/.ssh/id_rsa > ~/.ssh/id_rsa.pub
```


# Or, from the Holy Lab's Registry at WashU
# HolyLabRegistry

This registry allows you to use packages from HolyLab in Julia 0.7/1.x.

# Usage

If you're using at least Julia 1.1, then you can add this registry with

```
]registry add git@github.com:HolyLab/HolyLabRegistry.git
```

(The `]` enters Pkg mode when you type it at the REPL prompt, see https://docs.julialang.org/en/v1/stdlib/Pkg/.)

For earlier Julia versions, manually `git clone` this repository under `DEPOT_PATH/registries`. (Usually, `DEPOT_PATH = /home/username/.julia`)

Then, we can use lab private packages (or unregistered public ones) as if they are registered ones.

# To use git protocol in GitHub

This instruction is for Linux users and comes from https://help.github.com/articles/connecting-to-github-with-ssh/.
For windows users, you can get some information at https://gist.github.com/bsara/5c4d90db3016814a3d2fe38d314f9c23

0. Specific preperation for Windows

    - Create a folder at the root of your user home folder (Example: C:/Users/uname/) called .ssh.
    - Create the following files if they do not already exist (paths begin from the root of your user home folder):
    
        .ssh/config<br>
        .bash_profile<br>
        .bashrc<br>

1. Create a New SSH Key

    1.1 Generating a new SSH key at a local machine.
    - Open git bash and paste text below, substituting in your GitHub email address.
    ```
    $ ssh-keygen -t ecdsa -b 521 -C "your_email@example.com"
    ```
    Note: Around September 1, 2021, GitHub has added new security requirements for newly added RSA keys. Please see https://github.blog/2021-09-01-improving-git-protocol-security-github/ for more information.
    <!-- Note: It might have error when you install package from Holylab repository after you finished all steps, the error is "ERROR: failed to fetch from git@github.com". If you face this problem, it might be helpful to replace the command above by -->
    
    <!-- ```
    $ ssh-keygen -m PEM rsa -b 4096 -C "your_email@example.com"
    ```  -->
    
    - When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.
    ```
    Enter a file in which to save the key (/home/you/.ssh/id_rsa): [Press enter]
    ```

    - At the prompt, type a secure passphrase if you want.
    ```
    Enter passphrase (empty for no passphrase): [Type a passphrase]
    Enter same passphrase again: [Type passphrase again]
    ```

    1.2 Adding your SSH key to the ssh-agent
    - Start the ssh-agent in the background.
    ```
    $ eval "$(ssh-agent -s)"
    Agent pid 59566
    ```

    - Add your SSH private key to the ssh-agent
    ```
    $ ssh-add ~/.ssh/id_ecdsa
    ```
2. Setup SSH Authentication for Git Bash on Windows (Safe to skip for Linux)

    2.1 Configure SSH for Git Hosting Server
    - Add the following text to .ssh/config (.ssh should be found in the root of your user home folder):
    ```
    Host github.com<br>
    Hostname github.com<br>
    IdentityFile ~/.ssh/id_rsa
    ```
    
    2.2 Enable SSH Agent Startup Whenever Git Bash is Started
    - First, ensure that following lines are added to .bash_profile, which should be found in your root user home folder:
    ```
    test -f ~/.profile && . ~/.profile
    test -f ~/.bashrc && . ~/.bashrc
    ```    
    - Now, add the following text to .bashrc, which should be found in your root user home folder:
    ```
    # Start SSH Agent
    #----------------------------

    SSH_ENV="$HOME/.ssh/environment"

    function run_ssh_env {
      . "${SSH_ENV}" > /dev/null
    }

    function start_ssh_agent {
      echo "Initializing new SSH agent..."
      ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
      echo "succeeded"
      chmod 600 "${SSH_ENV}"

      run_ssh_env;

      ssh-add ~/.ssh/id_rsa;
    }

    if [ -f "${SSH_ENV}" ]; then
      run_ssh_env;
      ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
        start_ssh_agent;
      }
    else
      start_ssh_agent;
    fi
    ```
    
3. Adding a new SSH key to your GitHub account
    - Copies the contents of the id_rsa.pub file in the local machine to your clipboard
    - Go to GitHub site
    - In the upper-right corner of any page, click your profile photo, then click Settings.
    - In the user settings sidebar, click SSH and GPG keys.
    - Click New SSH key or Add SSH key.
    - In the "Title" field, add a descriptive label for the new key.
    - Paste your copied public key into the "Key" field
    - Click Add SSH key.


# For package developers

## Preparing your package before registering it

### Creating the directory and `Project.toml` file

You have two options:

- The manual way (generally not recommended):

  * Change your working directory to your development directory, typically `~/.julia/dev/`

  * Generate a package. Let your package name be 'MyPkg', then
    ```julia
    (v1.0) pkg> generate MyPkg
    ```
    This will generate a 'MyPkg' directory, 'Project.toml' including UUID, and sample source files.
    If you have already created the directory, this will cause an error.
    In this case, you can copy the 'Project.toml' file from a different package and edit it
    appropriately. Make sure you assign a new UUID, which can be generated with

- Using [PkgTemplates.jl](https://github.com/invenia/PkgTemplates.jl) (recommended):

  To create a new package and host it in your own GitHub account, use

  ```julia
  julia> using PkgTemplates

  julia> t = Template(ssh=true, plugins=[TravisCI()])  # creates a template for your personal account
  Template:
      → User: timholy
      → Host: github.com
      → License: MIT (Tim Holy 2019)
      → Package directory: /tmp/pkgs/dev
      → Minimum Julia version: v1.1
      → SSH remote: Yes
      → Commit Manifest.toml: No
      → Plugins: None

  julia> generate("MyPkg", t)
  # lots of output
  ```

  If you plan to host your new package on `HolyLab`, instead use

  ```julia
  julia> t = Template(user="HolyLab", ...)
  ```

### Adding dependent packages

- Change your working directory to `MyPkg`.

- Activate and add dependencies. Here we'll add `SubPkg1` and `SubPkg2` as dependencies:
  ```julia
  (v1.0)  pkg> activate .
  (MyPkg) pkg> add SubPkg1 SubPkg2
  ```
  This will add the dependent packages under the `[deps]` field in the 'Project.toml' and generate 'Manifest.toml' file.
  This 'Manifest.toml' file includes  specific versions of the dependent packages resolved according to your current environment.
  Usually, this file is excluded when you commit your package to a repository---if you created `MyPkg`
  using the manual method above, consider adding this to the `.gitignore` file.
  (PkgTemplates does this by default, see the "Commit Manifest.toml: No" line above.)

- Write whatever code and tests you need, commit them, and then push your package up to GitHub.

## Registering your package with HolyLabRegistry

### Using LocalRegistry

Check out a local copy of https://github.com/GunnarFarneback/LocalRegistry.jl.
Then:

- navigate to HolyLabRegistry, which for me is at `/home/tim/.julia/registries/HolyLabRegistry`
- update to the latest `master` branch
- check out a new branch, e.g., `git checkout -b teh/SomeNewPkg`
- start Julia and enter the following:
```julia
using LocalRegistry, SomeNewPkg
register(SomeNewPkg, "/home/tim/.julia/registries/HolyLabRegistry")
```
  where you replace the specific package name and path to the appropriate value on your system.
  This will add a new commit to the branch of HolyLabRegistry you just created
- Submit the branch as a PR to HolyLabRegistry
- Once the PR merges, from the HolyLabRegistry directory do
```
$ git checkout master
$ git pull
$ git branch -D teh/SomeNewPkg
```
- Push tags for the new release (`git tag -a vx.y.z` and then `git push --tags`)

### Manual approach (not recommended)

- Under the root directory of HolyLabRegistry, make directory including Compat.toml, Deps.toml, Package.toml and Versions.toml
  (To get started, you may want to copy related files from another existing directory in the registry)
- Edit those files appropriately:
  * Deps.toml : include all the dependencies (You can copy some lines from 'Project.toml' file of your package)
  * Package.toml : Package name, UUID, location of the repository
  * Versions.toml : git-tree-sha values according to version numbers.
    You can find this value with git command at the package root directory.
    ```
    $ git cat-file -p v0.1.0
    ```
    Then, You will see the value in the first line.
    If you just want to publish current 'master' branch, try this.
    ```
    $ git cat-file -p master
    ```
    In this case, every time master branch in the repository of the package is updated with new commit,
    you need to update this git-tree-sha value also in this registry.
    **Note: the git-tree-sha is different from the commit sha; in particular, don't use `git log` to get this value.**
- Add an entry for the package in Registry.toml file.
  As an example, add a below line if your package name is 'RFFT', directory name is also 'RFFT' and UUID is `3bd9afcd....`
  ```
  3bd9afcd-55df-531a-9b34-dc642dce7b95 = { name = "RFFT", path = "RFFT" }
  ```

## Accessing HolyLabRegistry in travis and appveyor tests

This is required only if your package uses other private packages.

- Include the following lines in the script section of the `.travis.yml` file in the root directory
  of your package (as an example, let your package name be 'RegisterFit')

  ```
  script:
  - if [[ -a .git/shallow ]]; then git fetch --unshallow; fi
  - julia -e 'using Pkg, LibGit2;
              user_regs = joinpath(DEPOT_PATH[1],"registries");
              mkpath(user_regs);
              all_registries = Dict("General" => "https://github.com/JuliaRegistries/General.git",
                                  "HolyLabRegistry" => "https://github.com/HolyLab/HolyLabRegistry.git");
              Base.shred!(LibGit2.CachedCredentials()) do creds
              for (reg, url) in all_registries
                  path = joinpath(user_regs, reg);
                  LibGit2.with(Pkg.GitTools.clone(url, path; header = "registry $reg from $(repr(url))", credentials = creds)) do repo end
              end
              end'
  - julia -e 'using Pkg; Pkg.build(); Pkg.test("RegisterFit"; coverage=false)'
  ```

  A similar script should be used with Appveyor (for testing on Windows).  However because multiline commands and variables are nightmarish in Windows it's recommended that you move the Julia command above into a separate script that gets called from `appveyor.yml`.  You can call the same script from `.travis.yml` as well to avoid code duplication.  See https://github.com/HolyLab/ImagineInterface for an example.

- Assign your private ssh key which is paired with a public key in your Github account to the package in the Travis site.
  * Copy the contents of the private key ('id_rsa' file generated in the 'To use git protocol in GitHub' section - not 'id_rsa.pub') in the local machine to your clipboard.
  * Go to the setup page of the package in the Travis site you want to make to access this registry. (You can get there by choosing the package in your Travis repositories, clicking ‘More options’ button on the upper right corner and selecting ‘setting’ menu.)
  * Assign the private key in the clipboard to the ‘SSH Key’ field.

## Tagging a new release

### In the package directory

- Edit the version number in `Project.toml`. All version numbers should be of the form `vX.X.X`, where each `X` is a number.
  See [semantic versioning](https://semver.org/) for guidelines about choosing version numbers.
- If your release has new version requirements for dependent packages, add those dependencies to the
  `[compat]` section. For example,
  ```
  [compat]
  DocStringExtensions = ">= 0.2"
  ```
- Commit the change(s) you made to `Project.toml`
- Create a git tag: `git tag -a vX.X.X` (where this matches the version number you used in `Project.toml`).
  In your editor, write a brief description of the new features of this release.
- Incorporate the changes in the `master` branch on GitHub, either by direct push or submitting a pull request.
- In preparation for the next step, execute `git cat-file -p vX.X.X` where again the version number matches your previous choices.

### In HolyLabRegistry

#### Using LocalRegistry

Just repeat the steps above for the initial registration, except that you don't have to specify the registry.

#### Manual approach (not recommended)

In the package's directory, update `Versions.toml` and, if necessary, `Compat.toml` and `Deps.toml`.
Use the sha from the `git cat-file` command above.

## Making a HolyLab package public on Github

- Make the package repo public by changing its Github settings.
- Be sure that you've done the same for any *dependencies* of the package.
- Submit a PR to this repo that changes the `url` field in `Project.toml` to use the https protocol instead of ssh (must also be done for any dependencies that you've made public).

# See also

- Creating a registry : https://discourse.julialang.org/t/creating-a-registry/12094

# Working with VS Code on PACE

When working with the Julia extension in VS Code, there are certain niceties that take some configuration when working remotely. The most prominent example is in-pane plotting. For this to work, you must enable X11 forwarding for your ssh connection/tunnel. The easiest way to do this is use a config file, which by default is located at `~/.ssh/config`. One example of an appropriate config file includes
```
Host atl*
  ForwardX11 yes
  ForwardAgent yes
  ForwardX11Trusted yes
  ProxyCommand ssh -X -o ForwardAgent=yes login-phoenix-slurm.pace.gatech.edu nc -w 120ms %h %p
```
This allows you to specify a node node to connect to (e.g. atl1-1-01-005-36-1) without specifying the proxy, X11 options, etc.

Using ssh key pairs, you can also eliminate the need to manually type in a password multiple times per connection. First, create a key pair and share the public key with PACE as usual, then modify the local ssh config file to include
```
# PACE
Host atl*
  ForwardX11 yes
  ForwardAgent yes
  ForwardX11Trusted yes
  User gburdell3
  IdentityFile C:\\PATH\\TO\\KEY\\id_rsa
  ProxyCommand ssh -X -o ForwardAgent=yes login-phoenix-slurm.pace.gatech.edu nc -w 120ms %h %p

Host login-phoenix-slurm.pace.gatech.edu
  HostName login-phoenix-slurm.pace.gatech.edu
  ForwardX11 yes
  ForwardAgent yes
  ForwardX11Trusted yes
  User gburdell3
  IdentityFile C:\\PATH\\TO\\KEY\\id_rsa
  ```
Note that that the double backslash file separator should only be necessary on Windows.
