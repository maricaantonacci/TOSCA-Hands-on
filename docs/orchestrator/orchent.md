### [◀](../README.md)

# Setup orchent

`orchent` is a command line tool to ease the interaction with the PaaS Orchestrator.

Admin and user guides are available [here](https://indigo-dc.gitbook.io/orchent/)

## Installation

```bash
orchent --version
```

## Configuration

orchent needs to use a valid (IAM) access token to authorize itself against the orchestrator.
For this purpose, `orchent` is integrated with `oidc-agent` so that you don't need to provide the token each time.
You need to configure the following environment variable in your `.bashrc`:

```bash
export ORCHENT_AGENT_ACCOUNT=dodas
```

Moreover, you need to configure the Orchestrator URL you want to connect to:
```bash
export ORCHENT_URL=https://dodas-paas.cloud.ba.infn.it/orchestrator
```

Then load the environment again:
```bash
source ~/.bashrc
```

## Test it!

Verify that everything is working fine as follows

- orchent has a simple way to test if the url points to an orchestrator:
      
    <pre>```
    orchent test
    ```</pre>
    
    The expected output is `looks like the orchent url is valid`. 

- list your deployments:
    
    <pre>```
    orchent depls -c me
    ```</pre>
    
    The expected output will show your deployment list (that can be empty of course: `retrieving deployment list:`)

    

