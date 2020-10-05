# Uploading Smart Contracts for Grid on Splinter

<!--
  Copyright (c) 2018-2020 Cargill Incorporated
  Licensed under Creative Commons Attribution 4.0 International License
  https://creativecommons.org/licenses/by/4.0/
-->

This procedure describes how to upload a new smart contract to a Splinter
circuit using Grid's command-line interface.

## Prerequisites

* An approved Splinter circuit with two or more member nodes.  See [Running
  Grid on Splinter]({% link docs/0.1/grid_on_splinter.md %}) and [Creating
  Splinter Circuits]({% link docs/0.1/creating_splinter_circuits.md %}).

* A fully qualified service ID for the scabbard service on this circuit, in
  the format <code><i>CircuitID</i>::<i>ServiceString</i></code>.
  This procedure uses the example service ID `01234-ABCDE::gsAA`.

  Tip: To determine the service ID, use `splinter circuit list` to display
  the circuit ID, then use that ID with `splinter circuit show` to see the
  service string for scabbard. For more information, see [Display the
  Circuit List and Details]({% link docs/0.1/creating_splinter_circuits.md
  %}#display-the-circuit-list-and-circuit-details).

* The Grid daemon's endpoint (URL and port) on one or both nodes.
  This procedure uses `https://localhost:8080`.

## Important Notes

The examples in this procedure use the node hostnames, container names, node
IDs, and URLs that are defined in the example Grid-on-Splinter docker-compose
file, `examples/splinter/docker-compose.yaml`. If you are not using this example
environment, replace these items with the actual values for your nodes.

## Procedure

### Connect to a Grid Node

1. Start a bash session in your node's `gridd` Docker container (such as
   `gridd-alpha`).  You will use this container to run Grid commands on the
   node (for example, `alpha-node-000`).

   ```
   $ docker exec -it gridd-alpha bash
   root@gridd-alpha:/#
   ```

2. Generate a secp256k1 key pair for the organization's agent on the alpha node.
   This key will be used to sign the Grid transactions that create organizations
   and set agent permissions.

   This example uses the base name `alpha-agent` to indicate that you will be
   acting as your organization's agent to add products from the alpha node.

   ```
   root@gridd-alpha:/# grid keygen alpha-agent`
   ```

   This command generates two files, `alpha-agent.priv` and `alpha-agent.pub`,
   in the `~/.grid/keys/` directory.

### Set Environment Variables

Set the following Grid environment variables to simplify entering the 
`grid` commands in this procedure.

1. Set `GRID_DAEMON_KEY` to the base name of your public/private key files
   (such as `alpha-agent`).

   ```
   root@gridd-alpha:/# export GRID_DAEMON_KEY="alpha-agent"
   ```

   **Note**: If you're using this example docker-compose environment, this
   variable is already defined for the `gridd-alpha` and `gridd-beta` containers.
   You can use `-k {BASENAME}` to override this variable on the command line.

1. Set `GRID_DAEMON_ENDPOINT` to the endpoint for the node's `gridd` container
   (such as `https://localhost:8080`).

   ```
   root@gridd-alpha:/# export GRID_DAEMON_ENDPOINT="https://localhost:8080"
   ```

   **Note**: If you're using this example docker-compose environment, this
   variable is already defined for the `gridd-alpha` and `gridd-beta` containers.
   You can use `--url {ENDPOINT}` to override this variable on the command
   line.

1. For Grid on Splinter: Set `GRID_SERVICE_ID` to the fully qualified service ID
   for the scabbard service on this circuit (such as `01234-ABCDE::gsAA`).

   ```
   root@gridd-alpha:/# export GRID_SERVICE_ID=01234-ABCDE::gsAA
   ```

   Tip: See the [Prerequisites](#prerequisites) for the `splinter` commands to
   display the components of the service ID.

### Create an Organization

1. Create a new organization, `myorg`.

   ```
   root@gridd-alpha:/# grid organization create \
   314156 myorg '123 main street' \
    --metadata gs1_company_prefixes=314156
   ```

   This command creates and submits a transaction to create a new Pike
   organization that is signed by the admin key. It also creates a new Pike
   agent with the “admin” role for the new organization (this agent’s public key
   is derived from the private key used to sign the transaction.) The service ID
   includes the circuit name and the scabbard service name for the alpha node.

1. XXX-REWRITE-XXX
   Create a new organization by specifying a unique organization ID, the
   organization's name and street address, and optional metadata (as key-value
   strings).

   This example uses the ID `314156`, the name `myorg` and imaginary address,
   and GS1-specific metadata to note that the ID is a GS1 company prefix.

   ```
   root@gridd-alpha:/# grid organization create 314156 myorg '123 main street' \
   --metadata gs1_company_prefixes=314156
   ```

   **Note**: This command doesn't display any output. Instead, check the log
   messages (or the terminal window where you started the Grid Docker
   environment) for the success or failure of this operation.

   This command creates and submits a transaction to create a new Pike
   organization with the data you supplied. The transaction is signed with your
   private key, as derived from the base name specified by `GRID_DAEMON_KEY` (or
   with the `-k` option on the command line).

   The `create` command also creates a new Pike agent using your public key (as
   derived from the same base name) as the agent ID.
   This command automatically assigns the admin role to the new agent, but does
   not make the agent active or enable any product permissions.

### Set Agent Permissions

1. Update the agent's permissions (Pike roles) to allow creating, updating, and
   deleting Grid products.

   ```
   root@gridd-alpha:/# grid \
   agent update 314156 $(cat ~/.grid/keys/alpha-agent.pub) --active \
   --role can_create_product \
   --role can_update_product \
   --role can_delete_product \
   --role admin
   ```

1. XXX-REWRITE-XXX
   Set the agent permissions (also called "Pike roles") to make the agent
   active and grant create, update, and delete permissions for product
   operations.

   ```
   root@gridd-alpha:/# grid agent update \
   314156 $(cat ~/.grid/keys/alpha-agent.pub) --active \
   --role can_create_product \
   --role can_update_product \
   --role can_delete_product \
   --role admin
   ```

   **Note**: You must specify `--role admin`, even though the previous command
   automatically assigned that role to the agent.

### Display Organizations and Agents

The Grid REST API provides the `/organization` and `/agent` endpoints to query
the distributed ledger for organization and agent information.

You can use `curl` to submit requests to the Grid REST API from the command
line. Note that the `curl` command is not available by default in the example
`gridd` container.

**Note**: Because REST API requests are handled by the scabbard service on an
existing circuit, a fully qualified service ID is required. 
If the `GRID_SERVICE_ID` environment variable is not set, see the
[Prerequisites](#prerequisites) for the `splinter` commands to determine
the service ID.

1. In the Grid node's `gridd` container, display the fully qualified service ID
   and copy it to use in the `curl` commands.

   ```
   root@gridd-alpha:/# echo $GRID_SERVICE_ID
   01234-ABCDE::gsAA
   ```

1. Request the list of organizations (from a system with `curl` installed).

   ```
   $ echo curl https://localhost:8080/organization?service_id=01234-ABCDE::gsAA
   ```

1. Request the list of agents.

   ```
   $ echo curl http://localhost:8080/agent?service_id=01234-ABCDE::gsAA
   ```

## Next Steps

Once you have an organization and one or more agents with product
permissions, you can define product schemas with Schema and create Product
records with Grid Product. For more information, see
[Using Grid Features]({% link docs/0.1/using_grid_features.md %}).












## ORIG CONTENT

### Demonstrate Smart Contract Deployment

The scabbard CLI enables deployment of custom smart contracts to existing
circuits.

1. Start a bash session in the `scabbard-cli-alpha` Docker container. You will
   use this container to send scabbard commands to `splinterd-alpha`.

   ```
   $ docker-compose -f examples/splinter/docker-compose.yaml run scabbard-cli-alpha bash
   root@scabbard-cli-alpha:/#
   ```

2. Set an environment variable to the circuit ID of the circuit that was created
   above.

   ```
   root@scabbard-cli-alpha:/# export CIRCUIT_ID=01234-ABCDE
   ```

3. Download the smart contract.

   `root@scabbard-cli-alpha:/# curl -OLsS https://files.splinter.dev/scar/xo_0.4.2.scar`

4. Create the contract registry for the new smart contract.

   ```
   root@scabbard-cli-alpha:/# scabbard cr create sawtooth_xo \
   --owners $(cat /root/.splinter/keys/gridd.pub) \
   -k gridd \
   -U 'http://splinterd-alpha:8085' \
   --service-id $CIRCUIT_ID::gsAA
   ```

5. Upload the smart contract.

   ```
   root@scabbard-cli-alpha:/# scabbard contract upload xo:0.4.2 \
   --path . \
   -k gridd \
   -U 'http://splinterd-alpha:8085' \
   --service-id $CIRCUIT_ID::gsAA
   ```

6. Create the namespace registry for the smart contract.

   ```
   root@scabbard-cli-alpha:/# scabbard ns create 5b7349 \
   --owners $(cat /root/.splinter/keys/gridd.pub) \
   -k gridd \
   -U 'http://splinterd-alpha:8085' \
   --service-id $CIRCUIT_ID::gsAA
   ```

7. Grant the appropriate contract namespace permissions.

   ```
   root@scabbard-cli-alpha:/# scabbard perm 5b7349 sawtooth_xo --read --write \
   -k gridd \
   -U 'http://splinterd-alpha:8085' \
   --service-id $CIRCUIT_ID::gsAA
   ```


8. Open a new terminal and connect to the `scabbard-cli-beta` container and add
   the circuit ID environment variable
   ```
    $ docker-compose \
    -f examples/splinter/docker-compose.yaml run scabbard-cli-beta bash
   root@scabbard-cli-beta:/#
   ```
   ```
   root@scabbard-cli-beta:/# export CIRCUIT_ID=01234-ABCDE
   ```

9. List all uploaded smart contracts.

   ```
   root@scabbard-cli-beta:/# scabbard contract list -U 'http://splinterd-beta:8085' --service-id $CIRCUIT_ID::gsBB
   NAME        VERSIONS OWNERS
   grid_product 1.0      <gridd-alpha public key>
   pike         0.1      <gridd-alpha public key>
   sawtooth_xo  1.0      <gridd-alpha public key>
   ```

10. Display the xo smart contract.

   ```
   root@scabbard-cli-beta:/# scabbard contract show sawtooth_xo:1.0 -U 'http://splinterd-beta:8085' --service-id $CIRCUIT_ID::gsBB
   sawtooth_xo 1.0
     inputs:
     - 5b7349
     outputs:
     - 5b7349
     creator: <gridd-alpha public key>
   ```
