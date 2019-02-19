# Connecting to a remote database

To connect to a remote montagu database you need to have postgresql installed.

```
sudo apt install postgresql postgresql-contrib
```

This installs postgresql and the `contrib` package which adds additional utilities. 

## VMs and databases

Each VM (uat, science and latest) contain a single database. 
 
* uat - Used for testing by the tech team, data tends to only be updated when required for testing new features.
* science - Used by the science team, contains draft reports and tends to be in front of production.
* latest - Nightly backup of science data.

## Connect to db

To connect to the remote database you need to know the following information:

* dbname - This is always `montagu`.
* hostname - This is always `support.montagu.dide.ic.ac.uk`.
* port - Specific to the VM you want to connect to, it can be found by checking the `dbport` configured via the [Vagrantfile](https://github.com/vimc/montagu-machine/blob/master/Vagrantfile) for the particular VM you want to connect to e.g. at the time of writing for uat this would be 15432.
* username - Either `readonly` or `import`, should be using `readonly` unless you have good reason to have elevated privs for the `import` user.
* password - Passwords on uat are the same as the username, for science and latest these can be found in the Montagu vault. To access these follow instructions from the [montagu-vault README](https://github.com/vimc/montagu-vault/blob/master/README.md) for accessing the vault. Passwords can then be found at `secret/database/production/users/<username>` and `secret/database/science/users/<username>`

Use `psql` to connect to the db.

```
psql -U <username> -h support.montagu.dide.ic.ac.uk -p <port> -d montagu
```

Enter the password when prompted.




 
