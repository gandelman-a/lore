# Testing with Vagrant

Clone hoist and navigate to its base directory:

```shell
$ git clone git@github.com:BonnyCI/hoist.git
$ cd hoist
```

To perform a full deploy:

```shell
$ vagrant up
```

To redeploy just the nodepool VM:

```shell
$ vagrant destroy nodepool
$ vagrant up nodepool
```

To inspect the zuul VM:

```shell
$ vagrant ssh zuul
$ # netstat, tcpdump, tail logs, etc.
$ logout
```

To test changes to the zuul role:

```shell
$ vagrant ssh bastion
$ sudo -i -u cideploy
$ /vagrant/tools/vagrant-run-ansible.sh --limit zuul
```

To tear down the entire stack when you're done:

```shell
$ vagrant destroy
```
