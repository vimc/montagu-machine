# dev machine

In order to spin up a local VM for testing deployment of montagu locally, run the following from the `montagu-machine` folder.

If you haven't already done so, set up Vagrant locally with 

```
sudo ./provision/setup-vagrant
``` 

To start the dev VM, run

```
./dev/deploy-dev-local <ref> 
```

This will tear down any dev VM which is already running. The montagu branch `<ref>` will be deployed in the VM 
from github. If no branch is specified, the 'latest' branch will be deployed.  

You can also optionally include `--prevent-deploy` to stop montagu actually being deployed on the VM. In this case, 
all the prerequisite set-up required by montagu will still be run.