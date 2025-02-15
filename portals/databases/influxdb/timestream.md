---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Amazon Timestream

Let's connect a nodejs app in one Amazon VPC with a Amazon Timestream managed InfluxDB database in another Amazon VPC. We’ll create an end-to-end encrypted Ockam Portal to InfluxDB.

To understand the details of how end-to-end trust is established, and how the portal works even though the two networks are isolated with no exposed ports, please read: “[<mark style="color:blue;">How does Ockam work?</mark>](../../../how-does-ockam-work.md)”

<figure><img src="../../../.gitbook/assets/influxdb-portal.png" alt=""><figcaption></figcaption></figure>

## Run

This example requires Bash, Git, AWS CLI. Please set up these tools for your operating system. In particular you need to [<mark style="color:blue;">login to your AWS account</mark>](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html).

{% hint style="info" %}
Amazon Timestream for InfluxDB was added very recently. To run this example, please install the latest version of AWS CLI.
{% endhint %}

Then run the following commands:

```bash
# Clone the Ockam repo from Github.
git clone --depth 1 https://github.com/build-trust/ockam && cd ockam

# Navigate to this example’s directory.
cd examples/command/portals/databases/influxdb/amazon_timestream/aws_cli

# Run the example, use Ctrl-C to exit at any point.
./run.sh
```

If everything runs as expected, you'll see the message: _The example run was successful 🥳_

## Walkthrough

The [<mark style="color:blue;">run.sh script</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh), that you ran above, and its [<mark style="color:blue;">accompanying files</mark>](https://github.com/build-trust/ockam/tree/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli) are full of comments and meant to be read. The example setup is only a few simple steps, so please take some time to read and explore.

### Administrator

* The [<mark style="color:blue;">run.sh script</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh) calls the [<mark style="color:blue;">run function</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh#L14) which invokes the [<mark style="color:blue;">enroll command</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh#L27) to create an new identity, sign into Ockam Orchestrator, set up a new Ockam project, make you the administrator of this project, and get a project membership [<mark style="color:blue;">credential</mark>](../../../reference/protocols/identities.md#credentials).
* The run function then [<mark style="color:blue;">generates two new enrollment tickets</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh#L36-L46). The tickets are valid for 10 minutes. Each ticket can be redeemed only once and assigns [<mark style="color:blue;">attributes</mark>](../../../reference/protocols/identities.md#credentials) to its redeemer. The [<mark style="color:blue;">first ticket</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh#L36-L37) is meant for the Ockam node that will run in Metrics Corp.’s network. The [<mark style="color:blue;">second ticket</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh#L44-L46) is meant for the Ockam node that will run in Datastream Corp.’s network.
* In a typical production setup an administrator or provisioning pipeline generates enrollment tickets and gives them to nodes that are being provisioned. In our example, the run function is acting on your behalf as the administrator of the Ockam project.
* The run function passes the enrollment tickets as variables of the run scripts provisioning [<mark style="color:blue;">Metrics Corp.'s network</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh#L51C41-L51C61) and [<mark style="color:blue;">Datastream Corp.'s network</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/run.sh#L56C44-L56C67).

### Metrics Corp

First, the `metrics_corp/run.sh` script creates a network to host the database:

* It [<mark style="color:blue;">creates a VPC</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L13-L17) and tags it.
* It [<mark style="color:blue;">creates an Internet gateway</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L19-L21) and attaches it to the VPC.
* It [<mark style="color:blue;">creates a route table</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L23-L24) and [<mark style="color:blue;">a route</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L25) to the Internet via the gateway.
* It [<mark style="color:blue;">creates a subnet</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L27-L32) and associates it with the route table.
* It [<mark style="color:blue;">creates a security group</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L34-L38) which allows:
  * TCP egress to the Internet.
  * Ingress to InfluxDB from within the subnet.
  * SSH ingress to provision EC2 instances.

Then, the `metrics_corp/run.sh` script creates a InfluxDB [<mark style="color:blue;">database</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L44-L63) using Timestream. Next the script creates an EC2 instance. This instance runs an Ockam TCP Outlet.

* It [<mark style="color:blue;">selects an AMI</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L68-L70).
* It then [<mark style="color:blue;">starts an instance using this AMI</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L77-L79) and a start script based on `run_ockam.sh` where:
  * [<mark style="color:blue;">`ENROLLMENT_TICKET`</mark> <mark style="color:blue;">is replaced by the enrollment ticket</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L75) created by the administrator and given as a parameter to `run.sh`.
  * [<mark style="color:blue;">`INFLUXDB_ADDRESS`</mark> <mark style="color:blue;">is replaced by the database address</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L76) that we previously saved.
* It [<mark style="color:blue;">tags the created instance</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L80) and [<mark style="color:blue;">waits for it to be available</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run.sh#L81).

When EC2 starts the instance, it executes the `run_ockam.sh` script:

* It installs the [Influxdb client](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run\_ockam.sh#L10-L11) and [configures it.](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run\_ockam.sh#L13-L16)
* It [<mark style="color:blue;">generates an InfluxDB auth token</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run\_ockam.sh#L21-L22) to send to Datastream Corp and saves it to file.
* It installs the [<mark style="color:blue;">`ockam`</mark> ](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run\_ockam.sh#L25-L26)command.
* It uses the [<mark style="color:blue;">enrollment ticket to create a default identity and make it a project member</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run\_ockam.sh#L41).
* It then [creates an Ockam node](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/metrics\_corp/run\_ockam.sh#L43-L59) with:
  * A TCP outlet.
  * An access control policy associated to the outlet. The policy authorizes only identities with a credential attesting to the attribute <mark style="background-color:yellow;">influxdb-inlet="true"</mark>.
  * A a relay that can forward TCP traffic to the TCP outlet.

### Datastream Corp

First, the `datastream_corp/run.sh` script creates a network to host the nodejs application:

* It [<mark style="color:blue;">creates a VPC</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L11-L12) and tags it.
* It [<mark style="color:blue;">creates an Internet gateway</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L15-L16) and attaches it to the VPC.
* It [<mark style="color:blue;">creates a route table</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L19) and [<mark style="color:blue;">a route</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L20) to the Internet via the gateway.
* It [<mark style="color:blue;">creates a subnet</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L23-L27) and associates it with the route table.
* It [<mark style="color:blue;">creates a security group</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L29-L36) that allows:
  * TCP egress to the Internet,
  * SSH ingress to provision EC2 instances.

Next, the script creates an EC2 instance. This instance runs an Ockam TCP Inlet.

* It [<mark style="color:blue;">selects an AMI</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L41-L43).
* It then [<mark style="color:blue;">starts an instance using that AMI</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L49-L51) and a start script based on `run_ockam.sh` in which the:
  * The variable [<mark style="color:blue;">`ENROLLMENT_TICKET`</mark> <mark style="color:blue;">is replaced by the enrollment ticket</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L48) created by the administrator and given as a parameter to `run.sh`.

When EC2 starts the instance, it executes the `run_ockam.sh` script:

* It installs [<mark style="color:blue;">`ockam`</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run\_ockam.sh#L10-L11) command.
* It uses the [<mark style="color:blue;">enrollment ticket is used to create a default identity and make it a project member</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run\_ockam.sh#L26).
* It then creates an Ockam node with:
  * A TCP inlet.
  * An access control [<mark style="color:blue;">policy associated with the inlet</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run\_ockam.sh#L39). The policy authorizes identities with a credential attesting to the attribute <mark style="background-color:yellow;">influxdb-outlet="true"</mark>.

Next `datastream_corp/run.sh` waits for the instance to be ready and [<mark style="color:blue;">provisions it using SSH</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L57-L69):

* It copies [<mark style="color:blue;">app.js and token.txt</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L57-L58) into the instance using SCP
* It then [<mark style="color:blue;">runs a script, using SSH</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L59-L69), which:
  * [<mark style="color:blue;">Installs nodejs</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L61).
  * [<mark style="color:blue;">Installs the InfluxDB client library</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L63).
  * [<mark style="color:blue;">Starts the nodejs application</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/run.sh#L67-L68).

Finally, the nodejs application is started:

* It [<mark style="color:blue;">connects to the Ockam inlet at localhost:8086</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/app.mjs#L8).
* It [<mark style="color:blue;">inserts a few system metrics into a bucket and retrieves t</mark><mark style="color:blue;">hem back</mark>](https://github.com/build-trust/ockam/blob/develop/examples/command/portals/databases/influxdb/amazon\_timestream/aws\_cli/datastream\_corp/app.mjs#L23-L92) to show that the connection with the InfluxDB database is working.

## Recap

<figure><img src="../../../.gitbook/assets/influxdb-portal.png" alt=""><figcaption></figcaption></figure>

We connected a nodejs app in one virtual private network with a InfluxDB database in another virtual private network over an end-to-end encrypted portal.

Sensitive business data in the InfluxDB database is only accessible to Metrics Corp. and Datastream Corp. All data is [<mark style="color:blue;">encrypted</mark>](../../../reference/protocols/secure-channels.md) with strong forward secrecy as it moves through the Internet. The communication channel is [<mark style="color:blue;">mutually authenticated</mark>](../../../reference/protocols/secure-channels.md) and [<mark style="color:blue;">authorized</mark>](../../../reference/protocols/access-controls.md). Keys and credentials are automatically rotated. Access to connect with InfluxDB can be easily revoked.

Datastream Corp. does not get unfettered access to Metrics Corp.’s network. It gets access only to query InfluxDB. Metrics Corp. does not get unfettered access to Datastream Corp.’s network. It gets access only to respond to queries over a tcp connection. Metrics Corp. cannot initiate connections.

All [<mark style="color:blue;">access controls</mark>](../../../reference/protocols/access-controls.md) are secure-by-default. Only project members, with valid credentials, can connect with each other. NAT’s are traversed using a relay and outgoing tcp connections. Metrics Corp. or Datastream Corp. don’t expose any listening endpoints on the Internet. Their networks are completely closed and protected from any attacks from the Internet.

## Cleanup

To delete all AWS resources:

```sh
./run.sh cleanup
```
