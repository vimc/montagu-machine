# dev machine

In order to spin up a local VM for testing deployment of montagu locally, run the following from the `montagu-machine` folder.

If you haven't already done so, set up Vagrant locally with 

```
./provision/setup-vagrant
``` 

To start the dev VM, run

```
./dev/deploy-dev-local <branchname>
```

This will tear down any dev VM which is already running. The montagu branch `<branchname>` will be deployed in the VM 
from github. If no branch is specified, the 'latest' branch will be deployed.  