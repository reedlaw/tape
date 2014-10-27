# WeaveUp Infrastructure Management

We've got the Ansible!

## Why?

Our old Capistrano system is too much of a ballache to add new tasks to.  Ansible is being used here for infrastructure mgmt since Capistrano isn't even the right tool for the job.

## Running

To generate hosts from capistrano hosts (only ip addresses supported): `./mkhosts.sh`

1. `ansible-galaxy install -r Rolefile` to install deps
2. `ssh-add` root keys from Passpack
3. `smashing_boxer -i hosts ansible deploy`

## The `smashing_boxer` tool

This tool provides a basic wrapper around the ansible scripts, and also contains a `qemu` module which is useful for testing ansible scripts locally.

Here's what a local test would look like using the tool

1. `smashing_boxer qemu create --name fe_test`
2. `smashing_boxer qemu start --name fe_test -p2255`
3. `smashing_boxer qemu add_ssh_key_to_agent`
4. `echo 'localhost:2255' >test_hosts`
5. `smashing_boxer ansible everything -i test_hosts`
 
Run `smashing_boxer -h` for a quick rundown of the tool's modules and options.

## Playbooks

### site

You can use site.example.yml to create a new site.yml for your project, based on omnibox. If site.yml lives in a directory different from where this repo is checked out you'll need `SMASHING_BOXER_PATH` set on the command line with this repo directory if you use the same include style that site.example.yml does.

Executing playbooks in other directories requires an env variable to point to the smashing boxer files.

Here's an example of how running this tool here with project-specific files looks for weaveup:

`SMASHING_BOXER_PATH=~/src/sb/smashing_boxer ansible-playbook -ihosts site.yml`

### omnibox

This is the setup right now.  This just puts everything on all the hosts, turning them all into single-machine deployments

## Roles

### General

General encapsulates basic stuff every box should have like unattended upgrades and swap space.

### Deployer User

This user needs to have some SSH keys so all the devs on the project can ssh to machines and do things.  Deployer user keys will live under roles/deployer\_user/files/keys.

### Web

Basic nginx installation, configured to work with unicorns running on the same box

This looks for the `fe_app` variable, and either configures nginx to serve `./public` from a rails app or `./dist` from an angular app at the root of the site.

### app\_server

Runs unicorns executing app code

### legacy

Used to migrate from old capistrano-managed boxes to the new infrastructure
