Notes & examples from Udemy's ansible course.
===================

Course Link :  https://www.udemy.com/mastering-ansible/

Prerequisites:

* A host with vagrant pre-installed
* Also requires the vagrant hostmanager plugin.

To Start Playing around:

```
# git clone git@github.com:gaumire/udemyanisble.git
# vagrant up
# vagrant ssh control
# cd /vagrant/
# start playing with ansible
```

To enable and activate all the guests after the guests are up
```
# vagrant ssh control
# cd /vagrant/
# ansible-playbook site.yml
```
