# Creating Products with Grid on Splinter

<!--
  Copyright (c) 2018-2020 Cargill Incorporated
  Licensed under Creative Commons Attribution 4.0 International License
  https://creativecommons.org/licenses/by/4.0/
-->

This procedure explains how to create and manage *Grid products*
(trade item data) using Grid's command-line interface.

[Grid Product]({% link docs/0.1/grid_product.md %}) provides the functionality
for managing product data, including the Product smart contract, a command-line
interface (the `grid product` subcommands), and a REST API interface (using the
`/product` and `/batches` endpoints). Grid Product is designed to interact with
[Schema]({% link docs/0.1/grid_schema_family_specification.md %}), which manages
product property schemas, and
[Pike]({% link docs/0.1/pike_transaction_family.md %}), which manages
organization IDs and agent permissions.

## Overview

This procedure starts with steps to [connect to a Splinter
node](#connect-to-a-splinter-node) and [set environment
variables](#set-environment-variables-optional) to simplify
entering the `grid` commands in the procedure.

Next, it explains how to [define a product](#define-the-product)
with a product YAML file, [create the product](#create-the-product) with
`grid product create`, and [display product
information](#display-product-information) with `grid product list`.

Finally, this procedure shows how to [update a product](#update-a-product)
(with `grid product update`) and [delete a product](#delete-a-product) (with
`grid product delete`) when it is no longer needed.

## Prerequisites

* A Splinter network with at least two nodes.

* An approved Splinter circuit that has two (or more) member nodes.

* The full service ID (also called the "full circuit ID") for this circuit, in
  the format `*circuitID*::*serviceID*`.
  This procedure uses the example ID `01234-ABCDE::xyZZ`.

  Tip: Use `splinter circuit list` to display the circuit and service IDs.
  In the example Docker environment, run this command in the node's
  `splinterd` container.

* The Grid daemon's endpoint (URL and port) on one or both nodes.
  The example environment uses `https://localhost:8080`.

* An organization ID and associated agent's public key for an existing Grid
  organization. This procedure assumes that you are the agent (your public key
  is used to identify the agent).

  Tip: You can use `curl` to request organization and agent information from
  the Grid REST API, as in these examples:

  `$ curl https://localhost:8080/organization?service_id=01234-ABCDE::xyZZ`

  `$ curl http://localhost:8080/agent?service_id=01234-ABCDE::xyZZ`

See [Running Hyperledger Grid on Splinter]({% link docs/0.1/grid_on_splinter.md
%}) for the procedures to [set up and run
Grid](/docs/0.1/grid_on_splinter.html#set-up-and-run-grid) in a set of Docker
containers, [create a
circuit](/docs/0.1/grid_on_splinter.html#create-a-circuit), and [create an
organization](/docs/0.1/grid_on_splinter.html#demonstrate-grid-smart-contract-functionality)
in the example Grid-on-Splinter environment.

## Procedure

**IMPORTANT**: The commands in this procedure show host names, IDs, Docker
container names, and other values from the example Grid-on-Splinter environment
that is defined by
[`grid/examples/splinter/docker-compose.yaml`](https://github.com/hyperledger/grid/blob/master/examples/splinter/docker-compose.yaml).
This file sets up the nodes `alpha-node-000` and `beta-node-000` and runs the
Grid and Splinter components in separate containers; these names appear in
example prompts and commands. (See [Running Hyperledger Grid on
Splinter]({% link docs/0.1/grid_on_splinter.md %}) for more information.)

If you are not using this example environment, replace these items with the
actual values for your environment when entering each command.

### Connect to a Splinter node

1. Connect to one of the Splinter nodes and start a bash session.

   For the example environment, use this command to connect to the `gridd-alpha`
   container and run Grid commands on `alpha-node-000`.

   ```
   $ docker exec -it gridd-alpha bash
   root@gridd-alpha:/#
   ```

### Set Up Your Environment

{:start="2"}

2. If necessary, generate your public/private key files in `$HOME/.grid/keys/`.

   ```
   root@gridd-alpha:/# grid keygen alpha-agent
   ```

   This example uses the alternate base name `alpha-agent` for the key files,
   because you will be acting as the organization's agent to add products.
   That is, your public key is used to identify the agent.

1. Set the following Grid environment variables to specify information for the
   `grid` commands in this procedure.

   a. Set `GRID_DAEMON_KEY` to the base name of your public/private key files.
      This environment variable replaces the `-k` option on the `grid` command
      line.

      ```
      root@gridd-alpha:/# export GRID_DAEMON_KEY="alpha-agent"
      ```

      **Note**: Although this variable has "daemon" in the name, it should
      reference the ***user key files*** in `$HOME/.grid/keys`, not the Grid
      daemon's key files in `/etc/grid/keys`.

   b. Set `GRID_DAEMON_ENDPOINT` to the Grid daemon's endpoint (URL and port),
      such as `http://localhost:8080`. This environment variable replaces the
      `-url` option on the `grid` command line.

      ```
      root@gridd-alpha:/# export GRID_DAEMON_ENDPOINT="http://localhost:8080"
      ```

   c. Set `GRID_SERVICE_ID` to the full circuit ID, in the format
      <code><i>circuitID</i>::<i>serviceID</i></code> (such as
      `01234-ABCDE::xyZZ`). This environment variable replaces the
     
     `--service_id` option on the `grid` command line.

      ```
      root@gridd-alpha:/# export GRID_SERVICE_ID="01234-ABCDE::xyZZ"
      ```

      Tip: See [Prerequisites](#prerequisites) for the commands to display
      circuit and service IDs.

### Define and Create a Product

This section has two steps. First, create a product definition file in YAML
format. Then run `grid product create` to create the product based on this
definition.

**Note**: Each product requires an organization ID (the owner) and at least one
agent with permission to create, update, and delete the product. See
[Prerequisites](#prerequisites) for the commands to display the existing
organizations and agents.

{:start="4"}

4. Create a product definition file, in YAML format, that specifies the
   following information:

   * Product identification type (such as GS1)
   * Product ID (such as a GTIN)
   * Owner (organization ID)
   * Set of product properties (each with a name, data type, and value
     that conforms to the data type)
     <br><br>

   For example, you can use `cat` to create the file `product.yaml`, using the
   following contents.

   ```
   root@gridd-alpha:/# cat > product.yaml
   - product_type: "GS1"
     product_id: "723382885088"
     owner: "314156"
     properties:
       - name: "species"
         data_type: "STRING"
         string_value: "tuna"
       - name: "length"
         data_type: "NUMBER"
         number_value: 22
       - name: "maximum_temperature"
         data_type: "NUMBER"
         number_value: 5
       - name: "minimum_temperature"
         data_type: "NUMBER"
         number_value: 0
   ```

   Tip: Enter CTRL-C to exit the file and save the contents.

1. Add the new product by using the `grid product create` command to specify the
   definition in the `product.yaml` file.

   ```
   root@gridd-alpha:/# grid product create product.yaml
   ```

   This command creates and submits a transaction to add the product data to the
   distributed ledger. If the transaction is successful, all other nodes in the
   circuit can view the product data.

   **Note**: This command does not display any output. Instead, check the log
   messages on the console (or the terminal window where you started the Grid
   Docker environment) for the success or failure of this operation.

### Display Product Information

{:start="6"}

6. List all existing products to verify that the new product has been added.

   ```
   root@gridd-alpha:/# grid product list
   Product Id: "723382885088"
    Product Type: "GS1"
    Owner: "314156"
    Properties:
	   Property Name: "species"
	    Data Type: "String"
	    Bytes Value: Some([])
	    Boolean Value: Some(false)
           Number Value: Some(0)
	    String Value: Some("tuna")
	    Enum Value: Some(0)
	    Struct Values: Some([])
	    Lat/Lon Values: Some(LatLong { latitude: 0, longitude: 0 })

           Property Name: "length"
	    Data Type: "Number"
           .
           .
           .
   ```

1. (Optional) You can connect to a different node and repeat the last two
   commands to verify that the product has been shared with all nodes on the
   circuit.

    a. Open a new terminal and connect to the other node's `gridd` container
       (such as `gridd-beta`).

      ```
      $ docker exec -it gridd-beta bash
      ```

    b. Set the `GRID_SERVICE_ID` environment variable to the full service ID
       for this node (such as `01234-ABCDE::xyBB`).

      ```
      root@gridd-beta:/# export GRID_SERVICE_ID="01234-ABCDE::xyBB"
      ```

    c. Display all products. The output should be the same as on the first node.

      ```
      root@gridd-beta:/# grid product list
      Product Id: "723382885088"
       Product Type: "GS1"
       Owner: "314156"
       Properties:
	      Property Name: "species"
	       Data Type: "String"
	       Bytes Value: Some([])
	       Boolean Value: Some(false)
              Number Value: Some(0)
	       String Value: Some("tuna")
	       Enum Value: Some(0)
	       Struct Values: Some([])
	       Lat/Lon Values: Some(LatLong { latitude: 0, longitude: 0 })

              Property Name: "length"
	       Data Type: "Number"
              .
              .
              .
      ```

### Update a Product

To update a product, you must be an agent for the product owner (the
organization that is identified in the product definition).

1. Modify the definition in the product YAML file (such as `product.yaml`).

   You don't have to use the same file that was used to create the product,
   but the file must specify the same information (product ID, type, and owner)
   and present the product properties in the same order.

1. Use the `update` subcommand for `grid product` to submit the changes
   to the distributed ledger.

   ```
   root@gridd-beta:/# grid product product.yaml
   ```

### Delete a Product

**IMPORTANT**: Deleting a product is a potentially hazardous operation that
must be done with care. Before you delete a product, make sure that no
member organizations on the circuit require the product data.

To delete a product, use the `delete` subcommand with the product ID and
product type (for example, ID `723382885088` and type `GS1`).

   ```
   root@gridd-beta:/# grid product delete 723382885088 GS1
   ```

Tip: Use `grid product list` to display the product ID and product type.
