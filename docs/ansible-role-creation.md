## Create new ansible role

## Preparation

```shell
source activate ansible-2.1
```

### Variables

```yaml
---

role_parent_dir: '~/projects/dev/roles'

role_short_name: 'nix'
role_long_name: 'ansible-role-{{ role_short_name }}''
role_description: 'An Ansible role to install and manage the Nix pack}{age manager'

role_github_user: 'cjsteel'
role_github_ssh_url: 'git@github.com:{{ role_github_user }}/{{ role_long_name }}.git'

role_galaxy_user: 'cjsteel'
role_galaxy_role_name: '{{ role_galaxy_user }}.{{ role_short_name }}

```

### Create repository

#### Locally

```git 
mkdir -p ~/projects/dev/roles
cd ~/projects/dev/roles
```

via cli using a python script

```shell
import argparse
from github import Github

#list of allowed prefixes for repos
# ansible-role - ansible-role
#p - project
#s - software
#r - ansible role
#h - hardware
#u - utilities/scripts etc.
#lib - libraries, reusable stuff
#feel free to add to this list as needed

repo_type_list = ['ansible-role', 'p', 's', 'h', 'u', 'lib']

token_file = '/home/user/.private/github_repo_token'
with open(token_file, 'r') as myfile:
    token=myfile.read().replace('\n', '')

# token = ""

#init argparser
parser = argparse.ArgumentParser(description='Create new github repo.')

parser.add_argument('-t', '--type', action='store', dest='type', required=True, choices=repo_type_list,
                    help='Initial character of repo name indicating type of repository.')

parser.add_argument('-n', '--name', action='store', dest='name', required=True,
                    help='Repo name after numbering and type indication.')

parser.add_argument('-d', '--description', action='store', dest='description', required=True,
                    help='Description of repository content.')

results = parser.parse_args()

#initialize github object
g = Github(token)

#init existing repo dict
existing_repo_list = {}
for element in repo_type_list:
    existing_repo_list[element] = []

#iterate through existing repos and add all numbers for a prefix to lists in prefix-dictionary
for repo in g.get_user().get_repos():
    repo_name = repo.name.encode('ascii','ignore')

    #check if repo name starts with one of the allowed prefixes
    if len([prefix for prefix in repo_type_list if repo_name.startswith(prefix)]) > 0:
        #find number after prefix but before first underscore
        repo_number = filter(str.isdigit, repo_name.split('_')[0])
        
#        if repo_number != '':
#            #if number is found, add to list of repo numbers for that prefix
#            existing_repo_list[[prefix for prefix in repo_type_list if repo_name.startswith(prefix)][0]].append(int(repo_number))

#check for spaces in repo name, which is a hassle
if ' ' in results.name:
    print("repo name cannot contain spaces, use _ (underscore) instead.")
    exit()

# change to check for suffix
#
#check if repo with that prefix exists from before and if it does, up number by 1
#if len(existing_repo_list[results.type])>0:
#    new_repo_number = max(existing_repo_list[results.type])+1
#else:
#    new_repo_number = 1

#form name for new repo
#new_repo_name = "{0}{1}_{2}".format(results.type, str(new_repo_number).zfill(3), results.name)
new_repo_name = "{0}-{1}".format(results.type, results.name)

#ask if this is the repo name we want
create_repo = raw_input(("Create repo {} ? yes for yes: ".format(new_repo_name))).strip()

#create repo
if create_repo == "yes":
    user = g.get_user()
    created_repo = user.create_repo(new_repo_name, description=results.description)
    print("repo {0} created for user {1}".format(created_repo.name, user.login))
else:
    print("exited without creating repo")
```

then

```shell
ansible-galaxy init cjsteel.nix
cd cjsteel.nix
git init
git-flow init -df
# run templates then...
echo "ADD README CONTENT" > README.md
git add README.md
git commit -m "Starting Out"
git remote add origin git@github.com:$repouser/$reponame.git
git push -u origin master
```

Create a description using the web interface for now, automate later.

## Templates

### Template resource

* [escaped jinja2 content](https://github.com/audreyr/cookiecutter/issues/11)

Copy .gitignore template

```shell
cp ../../../ace_templates/roles/.gitignore .
```

Copy imgs

```shell
cp -R ../../../ace_templates/roles/imgs .
```

Copy role README.md template

```shell
cp ../../../ace_templates/roles/README.md .
```

Copy role test templates

```shell
cp -R ../../../ace_templates/roles/tests .
```

Copy travis.yml template

```shell
cp ../../../ace_templates/roles/.travis.yml .
```

Copy role Vagrantfile template

```shell
cp ../../../ace_templates/roles/Vagrantfile .
```

## Edits

### Edit README.md

Replace `{{ short_role_name }}` with your roles short name and edit other headings as required.

```shell
cd ansible-role-{{ short_role_name }}
nano README.md
```

Content example:

```md

ansible-role-{{ short_role_name }}
=======================

An Ansible role to install and manage {{ short_role_name }}


Description
-----------

"**{{ short_role_name }}** is a 
```

### edit tests/vagrant.yml

```shell
nano tests/vagrant.yml
```

If all tests pass then clone form github to shortname

```shell
cd roles
# github
git clone git@github.com:cjsteel/ansible-role-{{ short_role_name }}.git
```

## 

### Add repository to Travis CI

to do 

## Add, commit, add origin and push

### git add

Ensure you are in your roles projects directory before runnning **git add**.

    cd roles/{{ short_role_name }}
Recursivly remove any temporary files in your directory. Here I use an alias that points to a shell script.

```shell
rmtmp
```

**git add** our projects files and directories (except those we listed in our .gitignore file earlier ).

We will use a **.** to indicate that we want to add everything and that we wish to do it recursivly. In the future you may want to add a single file at a time in order to avoid accidental commits.

```shell
git add .
git status
```


### git commit

If everything is OK, we will commit all of our changes to our repository.

```shell
git commit -m 'initial commit of ansible role scaffolding and boilerplate'
```


### git remote add origin

```shell
git remote add origin git@ace-git-1.cbrain.mcgill.ca:csteel/ansible-role-{{ short_role_name }}.git	
```


### git push -u origin master

Finally we push to our master

```shell
git push -u origin master
```

## git clone

Now we will clone the project and delete our original working directory only this time we will give it the short name **{{ short_role_name }}** rather than the default name **ansible-role-{{ short_role_name }}**.

```shell
cd ..
git clone git@ace-git-1.cbrain.mcgill.ca:csteel/ansible-role-{{ short_role_name }}.git {{ short_role_name }}
```

## mr

If you are running myrepos now is a great time to register your new repository

```shell
cd {{ short_role_name }}
mr register
mr status
```

test

```shell
vagrant up
```

Great, your all set to start coding!

## Dependancies

Rough edit below:

## ansible-role-ensure_dirs

Using the role [ansible-role-ensure_dirs]() as a dependancy requires that the calling role have the following:

### meta/main.yml

`meta/main.yml` is where we need to describe the dependancy and is where we pass our variables from the local role to [ansible-role-ensure_dirs]()

```yml

```



### define local and remote deployment users.

```
rolename_remote_deployment_user: msmith
rolename_local_deployment_user: vkundra
```



### define remote and local resource directories

```shell
minc_ensure_dirs_on_remote:

  minc_remote_resource_dir:

    state          : 'directory'
    path           : '{{ minc_remote_resource_directory }}"
    owner          : '{{ minc_deployment_user }}'
    group          : '{{ minc_deployment_user }}'
    mode           : '0755'


minc_ensure_dirs_on_local:

  minc_local_resource_dir:

    state          : 'directory'
    path           : '{{ minc_local_resource_directory }}'
    owner          : '{{ minc_deployment_user }}'
    group          : '{{ minc_deployment_user }}'
    mode           : '0755'
```

## License

MIT

## Author Information

Christopher Steel
Systems Administrator
McGill Centre for Integrative Neuroscience
Montreal Neurological Institute
McGill University
3801 University Street
Montréal, QC, Canada H3A 2B4
Tel. No. +1 514 398-2494
E-mail: christopherDOTsteel@mcgill.ca
[MCIN](http://mcin-cnim.ca/), [theneuro.ca](http://theneuro.ca)

