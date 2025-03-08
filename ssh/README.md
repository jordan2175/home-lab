# SSH / SSHD Configuration Options

## Generate SSH Keys

ssh-keygen -t ed25519 -C "Username Default"
ssh-keygen -t ed25519 -C "Ansible Default"

## Copy keys to server

This can be done by hand, but the tool makes it easier 

ssh-copy-id -i ~/.ssh/id_ed25519.pub 10.1.2.3
ssh-copy-id -i ~/.ssh/ansible.pub 10.1.2.3


## Use specific key

ssh -i ~/.ssh/ansible 10.1.2.3


## Cache passphrase for private key

ssh-agent = agent that can cache passphrase

This command will start the ssh-agent and return the process id (pid)
eval $(ssh-agent)

We can then see this by
ps aux | grep pid-from-last-command or
ps aux | grep ssh-agent

This command will add the key to the ssh-agent. But it will only be there for the life of that terminal window. This will ask you for the ssh password for your private key.
ssh-add

This command will start the ssh-agent and add the current id to it
alias ssha='eval $(ssh-agent) && ssh-add'
