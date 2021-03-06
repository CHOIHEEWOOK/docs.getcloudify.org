---
layout: bt_wiki
title: Google Cloud Plugin
category: Plugins
draft: false
weight: 100
---
{{% gsSummary %}} {{% /gsSummary %}}

The GCP plugin allows users to use Cloudify to manage cloud resources on GCP. See below for currently supported resource types.

Be aware that some services and resources vary in availability between regions and accounts.


# Plugin Requirements

* Python versions:
  * 2.7.x
* [GCP](https://cloud.google.com/) account


# Compatibility

The GCP plugin uses the official [Google API Python Client](https://github.com/google/google-api-python-client).


# Terminology

* Region refers to a general geographical area, such as "Central Europe" or "East US".
* Zone refers to a distinct area within a region. Zones are usually referred to as '{region}-{zone}, i.e. 'us-east1-b' is a zone within the reigon 'us-east1'.


# Types

The following are [node type]({{< relref "blueprints/spec-node-types.md" >}}) definitions. Nodes describe resources in your cloud infrastructure. For more information, see [node type]({{< relref "blueprints/spec-node-types.md" >}}).

## Common Properties

All cloud resource nodes have common properties:

**Properties**

Every time you manage a resource with Cloudify,
it creates one or more connections to the GCP API.
You specify the configuration for these clients using the `gcp_config` property.
It should be a dictionary, with the following values:

  * `project` the name of your project on GCP.
  * `zone` the default zone which will be used,
    unless overridden by a defined zone/subnetwork.
  * `auth` the JSON key file provided by GCP.
    Can either be the contents of the JSON file, or a file path.
    This should be in the format provided by the GCP credentials JSON export (https://developers.google.com/identity/protocols/OAuth2ServiceAccount#creatinganaccount)
  * (optional) `network` the default network to place network-scoped nodes in.
    The default network (`default`) will be used if this is not specified.

Example
{{< gsHighlight  yaml  >}}
...
node_types:
  my_vm:
    type: cloudify.gcp.nodes.Instance
    properties:
      image_id: <GCP image ID>
      gcp_config:
        project: a-gcp-project-123456
        zone: us-east1-b
        auth: <GCP auth JSON file>
{{< /gsHighlight >}}


## Using Existing Resources

Many Cloudify GCP types have a property named `use_external_resource`, which defaults to `false`. When set to `true`, the plugin will apply different semantics for each of the operations executed on the relevant node's instances:

  If `use_external_resource` is `true`, then the required properties for that type (`name`, possibly `region` or `zone`) will be used to look up an existing entity in the GCP project.
  If the entity is found, then its data will be used to popluate the cloudify instance's attributes (`runtime_properties`). If it is not found then the blueprint will fail to deploy.


This behavior is common to all resource types which support `use_external_resource`:

 * `create` If `use_external_resource` is true, the GCP plugin will check if the resource is available in your account. If no such resource is available, the operation will fail, if it is available, it will assign the resource details to the instance `runtime_properties`.
 * `delete` If `use_external_resource` is true, the GCP plugin will check if the resource is available in your account. If no such resource is available, the operation will fail, if it is available, it will unassign the instance `runtime_properties`.


## Runtime Properties

See section on [runtime properties](http://cloudify-plugins-common.readthedocs.org/en/3.3/context.html?highlight=runtime#cloudify.context.NodeInstanceContext.runtime_properties)

Most node types will write a snapshot of the `resource` information from GCP when the node creation has finished (some, e.g. DNSRecord don't correspond directly to an entity in GCP, so this is not universal).


# Node Types

## cloudify.gcp.nodes.Address
**Derived From:** cloudify.gcp.nodes.GlobalAddress

A GCP Address. This can be connected to an Instance using the `cloudify.gcp.relationships.instance_connected_to_ip` relationship type



**Properties:**


  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `region`
    Region to place the Address in. If not provided it defaults to the value in `gcp_config` (which defaults to 'default').

    *default:* 





## cloudify.gcp.nodes.BackendService
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A group of Instances (contained within InstanceGroups) which can be used as the backend for load balancing.



**Properties:**


  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `additional_settings`
    Optional additional settings. Possible fields in dictionary are: port_name, protocol, timeout_sec.

    *default:* {}
  * `health_check`
    URL of a health check assigned to this backend service.

    *type:* string
    *default:* 
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true) or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False
  * `name`
    Optional health check name. By default it will be backend service id.

    *type:* string
    *default:* 





## cloudify.gcp.nodes.DNSAAAARecord
**Derived From:** cloudify.gcp.nodes.DNSRecord

`AAAA` type DNSRecord



**Properties:**


  * `type`
    

    *default:* AAAA





## cloudify.gcp.nodes.DNSMXRecord
**Derived From:** cloudify.gcp.nodes.DNSRecord

`MX` type DNSRecord



**Properties:**


  * `type`
    

    *default:* MX





## cloudify.gcp.nodes.DNSNSRecord
**Derived From:** cloudify.gcp.nodes.DNSRecord

`NS` type DNSRecord



**Properties:**


  * `type`
    

    *default:* NS





## cloudify.gcp.nodes.DNSRecord
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

Corresponds to a particular subdomain (or `@` for the root) and record-type in the containing DNSZone.

e.g. the `A` record for `special_service.getcloudify.org`

A number of convenience types are provided which update the default type (see DNSAAAARecord, DNSMXRecord, DNSTXTRecord, DNSNSRecord)



**Properties:**


  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `type`
    The type of this DNS record. Only one record of each type is allowed with the same name within a zone.

    *type:* string
    *default:* A
  * `name`
    The subdomain. This will be prepended to the DNSZone's dns_name to produce the full domain name for this record. Defaults to the instance ID.

    *type:* string
    *default:* 
  * `resources`
    List of resources which will form this record. (can be augmented using relationships
      cloudify.gcp.relationships.dns_record_connected_to_instance
      and
      cloudify.gcp.relationships.dns_record_connected_to_ip
      )

    *default:* []
  * `ttl`
    DNS entry Time To Live

    *type:* integer
    *default:* 86400



### Example

{{< gsHighlight  yaml  >}}

www:
  type: cloudify.gcp.nodes.DNSRecord
  properties:
    resources: [10.11.12.13, 8.9.10.11]
  relationships:
    - type: cloudify.gcp.relationships.dns_record_contained_in_zone
      target: my_zone

mx:
  type: cloudify.gcp.nodes.DNSMXRecord
  properties:
    name: mail
  relationships:
    - type: cloudify.gcp.relationships.dns_record_contained_in_zone
      target: my_zone
    - type: cloudify.gcp.relationships.dns_record_connected_to_instance
      target: my_instance

{{< /gsHighlight >}}

The DNSRecord type can be connected to an instance or directly to an IP. In this case the (associated) public IP will be added to the list of resources.



## cloudify.gcp.nodes.DNSTXTRecord
**Derived From:** cloudify.gcp.nodes.DNSRecord

`TXT` type DNSRecord



**Properties:**


  * `type`
    

    *default:* TXT





## cloudify.gcp.nodes.DNSZone
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A Cloud DNS zone.

Represents a particular DNS domain which you wish to manage through Google Cloud DNS.
DNS nameservers can vary between different DNSZones. In order to find the correct nameserver entries for your domain, use the `nameServers` attribute from the created zone.



**Properties:**


  * `dns_name`
    (fully qualified) domain name of the zone. Defaults to the instance ID.

    *type:* string
    *default:* 
  * `additional_settings`
    Additional settings

    *default:* {}
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true)  or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False
  * `name`
    (internal) name of the zone. Defaults to the instance ID.

    *type:* string
    *default:* 
  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}



### Example

{{< gsHighlight  yaml  >}}

my_zone:
  type: cloudify.gcp.nodes.DNSZone
  properties:
    dns_name: getcloudify.org.

{{< /gsHighlight >}}

The `dns_name` supplied must be a fully-qualified domain name with the trailing dot. The output attributes (`runtime_properties`) will include a key `nameServers` which contains the list of nameservers that should be supplied as nameservers with the domain registrar.



## cloudify.gcp.nodes.ExternalIP
**Derived From:** [cloudify.nodes.VirtualIP]({{< relref "blueprints/built-in-types.md" >}})

Use this together with the `cloudify.gcp.relationships.instance_connected_to_ip` if you want the Instance to have an ephemeral external IP



**Properties:**


  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `ip_address`
    Address of this external IP. This should be address of already existing, unattached static IP. It will be used only if "use_external_resource" is set to true.

    *type:* string
    *default:* 
  * `use_external_resource`
    Indicate whether the resource exists or if Cloudify should create the resource. If set to true, this node will be static IP, otherwise ephemeral IP.

    *type:* boolean
    *default:* False





## cloudify.gcp.nodes.FirewallRule
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A GCP FirewallRule.

This describes allowed traffic directed to either the whole of the specified network, or to Instances specified by matching tags.



**Properties:**


  * `sources`
    List of CIDR formatted ranges and instance tags which
    will be permitted to connect to targets by this rule
    e.g.:
      - 10.100.101.0/24
      - a-tag

    *required* None
  * `additional_settings`
    Additional setting for firewall

    *default:* {}
  * `name`
    Optional security group name. By default it will be network name plus node name.

    *default:* 
  * `allowed`
    Dictionary of allowed ports per protocol, in the form
      protocol: [port, ...]
    If no ports are specified then all ports are opened for that protocol eg:
      tcp: 80, 443
      udp:

    *required* None
  * `gcp_config`
    A dictionary of values to pass to authenticate with the Google Cloud Platform API.

    *default:* {}
  * `target_tags`
    List of target tags this rule should apply to. If no tags are specified, it will apply to all instances in the network

    *default:* []
  * `use_external_resource`
    Indicate whether the resource exists or if Cloudify should create the resource.

    *type:* boolean
    *default:* False



### Example

{{< gsHighlight  yaml  >}}

allow_ssh:
  type: cloudify.gcp.nodes.FirewallRule
  properties:
    sources: [0.0.0.0/0]
    allowed:
      tcp: [22]

allow_http_to_http_tag:
  type: cloudify.gcp.nodes.FirewallRule
  properties:
    sources: [0.0.0.0/0]
    allowed:
      tcp: [80]
    target_tags: [http]

http_instance:
  type: cloudify.gcp.nodes.Instance
  properties:
    tags: [http]
    ...

{{< /gsHighlight >}}





## cloudify.gcp.nodes.GlobalAddress
**Derived From:** [cloudify.nodes.VirtualIP]({{< relref "blueprints/built-in-types.md" >}})

A GCP GlobalAddress.

GlobalAddress can only be used together with GlobalForwardingRule. If you want to connect a static IP to an Instance, use StaticIP instead.



**Properties:**


  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `additional_settings`
    Additional setting for static ip

    *default:* {}
  * `use_external_resource`
    Indicate whether the resource exists or if Cloudify should create the resource. If set to true, this node will be already existing static IP address, otherwise it will be reserved static IP address.

    *type:* boolean
    *default:* False
  * `name`
    Optional static ip name. By default it will be static ip id.

    *type:* string
    *default:* 





## cloudify.gcp.nodes.GlobalForwardingRule
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A GCP GlobalForwardingRule.

Can only be used in conjunction with a GlobalAddress to set up HTTP and HTTPS forwarding.



**Properties:**


  * `port_range`
    Port number used by this forwarding rule. If packets are redirected to HTTP proxy, then possible values are 80 and 8080, in case of HTTPS proxy the only accepted value is 443.

    *type:* string
    *default:* 80
  * `additional_settings`
    Additional setting for ssl certificate

    *default:* {}
  * `name`
    Optional global forwarding rule name. By default it will be global forwarding rule id.

    *type:* string
    *default:* 
  * `target_proxy`
    URL of a target proxy (http or https) that will receive traffic coming from specified IP address.

    *type:* string
    *default:* 
  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `ip_address`
    IP address associated with this forwarding rule. This address should be reserved earlier.

    *type:* string
    *default:* 
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true) or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False





## cloudify.gcp.nodes.HealthCheck
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A GCP HealthCheck.

This describes a method that a TargetProxy can use to verify that particualr backend Instances are functioning. Backends which fail the health check verification will be removed from the list of candidates.



**Properties:**


  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `additional_settings`
    Optional additional settings. Possible fields in dictionary are: port, request_path, timeout_sec, check_interval_sec, healthy_threshold, unhealthy_threshold.

    *default:* {}
  * `health_check_type`
    This field indicates if this health check is a HTTP or HTTPS based health check. Possible values are: 'http' and 'https'.

    *type:* string
    *default:* http
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true) or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False
  * `name`
    Optional health check name. By default it will be health check id.

    *type:* string
    *default:* 





## cloudify.gcp.nodes.Image
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A stored image which can be used as the base for newly created Instances.



**Properties:**


  * `image_name`
    Name to use for the image. Defaults to the instance ID.

    *default:* 
  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `additional_settings`
    Additional setting for image

    *default:* {}
  * `image_path`
    The (local system) path to the image file which will be uploaded.

    *default:* 
  * `use_external_resource`
    Indicate whether the resource exists or if Cloudify should create the resource.

    *type:* boolean
    *default:* False





## cloudify.gcp.nodes.Instance
**Derived From:** [cloudify.nodes.Compute]({{< relref "blueprints/built-in-types.md" >}})

A GCP Instance (i.e. a VM).


**Properties:**


  * `scopes`
    Optional scopes. If not will set by default:  'https://www.googleapis.com/auth/devstorage.read_write', 'https://www.googleapis.com/auth/logging.write'

    *default:* []
  * `instance_type`
    The instance's type. All available instance types can be found here:  https://cloud.google.com/compute/docs/machine-types

    *type:* string
    *default:* n1-standard-1
  * `name`
    Optional instance name. By default it will be instance id.

    *type:* string
    *default:* 
  * `zone`
    Optional zone name. If not given, this instance will be deployed in default zone.

    *type:* string
    *default:* 
  * `tags`
    Optional tags. If not given, this instance will have a tag only with its name.

    *type:* string
    *default:* 
  * `external_ip`
    Should the Instance be created with an externally-accessible IP address. This will be an ephemeral IP. If you would like to use an IP address which can be transferred to another Instance then connect this Instance to an `Address` node using the `cloudify.gcp.relationships.instance_connected_to_ip` relationship.

    *type:* boolean
    *default:* False
  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `block_project_ssh_keys`
    Disable project-wide ssh keys for this Instance

    *type:* boolean
    *default:* False
  * `image_id`
    The ID of the image in your GCP account.

    *type:* string
    *default:* {}
  * `additional_settings`
    Additional instance settings.

    *default:* {}
  * `startup_script`
    A script which will be run when the Instance is first started example:
      type: string
      script: |
        yum install some stuff
        systemctl start it
    or:
      type: file
      script: <path to script file>

    *default:* 
  * `can_ip_forward`
    Is the VM allowed to send packets with source address different to its own?

    *type:* boolean
    *default:* False
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true)  or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False



### Example

{{< gsHighlight  yaml  >}}

my_gcp_instance:
  type: cloudify.gcp.nodes.Instance
  properties:
    image_id: http://url.to.your.example.com/image
    instance_type: n1-standard-1
    gcp_config:
      project: your-project
      network: default
      zone: us-east1-b
      auth: path_to_auth_file.json

{{< /gsHighlight >}}

This example includes shows adding additional parameters, tagging an instance name, and explicitly defining the gcp_config.




## cloudify.gcp.nodes.InstanceGroup
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A GCP InstanceGroup.
This is used to configure failover systems. InstanceGroups can be configured to scale automatically based on load, and will replace failing Instances with freashly started ones.


**Properties:**


  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `additional_settings`
    Additional setting for instance group

    *default:* {}
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true) or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False
  * `name`
    Optional instance name. By default it will be instance group id.

    *type:* string
    *default:* 
  * `named_ports`
    A list of named ports defined for this instance group, the expected format is: [{name: 'name', port: 1234}, ... ].

    *default:* []





## cloudify.gcp.nodes.KeyPair
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

An SSH key-pair which will be uploaded to any Instances connected to it via `cloudify.gcp.relationships.instance_connected_to_keypair`.

Unlike other cloud providers, users are dynamically created on Instances based on the username specified by the uploaded SSH key, so the public key text must include a username in the comment section (keys generated using `ssh-keygen` have this by default).



**Properties:**


  * `private_key_path`
    The path where the key should be saved on the machine. If this is a bootstrap process, this refers to the local computer. If this will run on the manager, this will be saved on the manager.

    *type:* string
    *default:* 
  * `public_key_path`
    The path to read from existing public key.

    *type:* string
    *default:* 
  * `user`
    The user account for this key. A corresponding user account will be created by GCP when the key is added to the Instance. This must be supplied for a non-external resource key. See https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys

    *type:* string
    *default:* 
  * `gcp_config`
    A dictionary of values to pass to authenticate with the Google Cloud Platform API.

    *default:* {}
  * `use_external_resource`
    Indicate whether the resource exists or if Cloudify should create the resource.

    *type:* boolean
    *default:* False





## cloudify.gcp.nodes.Network
**Derived From:** [cloudify.nodes.Network]({{< relref "blueprints/built-in-types.md" >}})

A GCP Network. This supports either auto-assigned or manual subnets. Legacy networks are not supported. See the GCP Manager and Networks section below if you plan to run a cloudify manager on GCP.



**Properties:**


  * `gcp_config`
    A dictionary of values to pass to authenticate with the Google Cloud Platform API.

    *default:* {}
  * `auto_subnets`
    Whether to use the GCP "autoCreateSubnetworks" feature (see https://cloud.google.com/compute/docs/subnetworks#networks_and_subnetworks)

    *default:* True
  * `additional_settings`
    Additional setting for network

    *default:* {}
  * `name`
    Optional Network name. The instance ID will be used by default.

    *default:* 
  * `use_external_resource`
    Indicate whether the resource exists or if Cloudify should create the resource.

    *type:* boolean
    *default:* False



### Example

{{< gsHighlight  yaml  >}}

my_net:
  type: cloudify.gcp.nodes.Network

{{< /gsHighlight >}}





## cloudify.gcp.nodes.Route
**Derived From:** [cloudify.nodes.Router]({{< relref "blueprints/built-in-types.md" >}})

A defined route, which will be added to the specified network.
If tags are specified, it will only be added to Instances matching them.



**Properties:**


  * `dest_range`
    The outgoing range that this route will handle

    *required* None
  * `priority`
    The routing table priority for this route. Routes with lower priority numbers will be chosen first if more than one route with a matching prefix of the same length.

    *default:* 1000
  * `additional_settings`
    Additional setting for firewall

    *default:* {}
  * `next_hop`
    The Instance, IP or VpnTunnel which will handle the matching packets

    *default:* 
  * `name`
    Optional Route name. The instance ID will be used by default.

    *default:* 
  * `tags`
    Instance tags that this route will be applied to

    *default:* []
  * `gcp_config`
    A dictionary of values to pass to authenticate with the Google Cloud Platform API.

    *default:* {}





## cloudify.gcp.nodes.SecurityGroup
**Derived From:** [cloudify.nodes.SecurityGroup]({{< relref "blueprints/built-in-types.md" >}})

A virtual SecurityGroup.

Google Cloud Platform has no entity equivalent to a Security Group on AWS or OpenStack, so as a convenience Cloudify includes a virtual one. It is implemented behind the scenes using a specially constructed tag and a number of FirewallRules.



**Properties:**


  * `rules`
    List of FirewallRules which will form this SecurityGroup. Only the `sources:` and `allowed:` fields should be supplied (see FirewallRule properties for details).

    *default:* []
  * `gcp_config`
    A dictionary of values to pass to authenticate with the Google Cloud Platform API.

    *default:* {}
  * `name`
    Optional security group name. By default it will be network name plus node name.

    *default:* 





## cloudify.gcp.nodes.SslCertificate
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A TLS/SSL certificate and key. This will be used by a HTTPS TargetProxy to provide authenticated encryption for connecting users.



**Properties:**


  * `private_key`
    Dictionary describing private key in PEM format used to generate this SSL certificate. Expected format is:
      type: text|file
      data: Private key in PEM format if text, otherwise path to a file with private key

    *default:* {}
  * `name`
    Optional SSL certificate name. By default it will be SSL certificate id.

    *type:* string
    *default:* 
  * `certificate`
    Certificate (self-signed or obtained from CA) in PEM format. Expected format is:
      type: text|file
      data: Certificate in PEM format if text, otherwise path to a file with certificate

    *default:* {}
  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `additional_settings`
    Additional setting for target proxy

    *default:* {}
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true) or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False





## cloudify.gcp.nodes.StaticIP
**Derived From:** cloudify.gcp.nodes.GlobalAddress

Alias for GlobalAddress for backward compatibility.



**Properties:**







## cloudify.gcp.nodes.SubNetwork
**Derived From:** [cloudify.nodes.Subnet]({{< relref "blueprints/built-in-types.md" >}})

A GCP Subnetwork. Must be connected to a Network using `cloudify.gcp.relationships.contained_in_network`.

Only networks with the `auto_subnets` property disabled can be used.



**Properties:**


  * `subnet`
    The subnet, denoted in CIDR form (i.e. '10.8.0.0/20') Subnets must be unique and non-overlapping within a project. See https://cloud.google.com/compute/docs/subnetworks#networks_and_subnetworks

    *type:* string
    *default:* 
  * `region`
    The region this subnet is in. See https://cloud.google.com/compute/docs/regions-zones/regions-zones

    *type:* string
    *default:* 
  * `use_external_resource`
    Indicate whether the resource exists or if Cloudify should create the resource.

    *type:* boolean
    *default:* False
  * `name`
    Optional SubNetwork name. The instance ID will be used by default.

    *default:* 
  * `gcp_config`
    A dictionary of values to pass to authenticate with the Google Cloud Platform API.

    *default:* {}



### Example

{{< gsHighlight  yaml  >}}

my_net:
  type: cloudify.gcp.nodes.Network
  properties:
    auto_subnets: false

my_subnet:
  type: cloudify.gcp.nodes.SubNetwork
  properties:
    subnet: 10.8.0.0/20
  relationships:
    - type: cloudify.gcp.relationships.contained_in_network
      target: my_net

my_instance:
  type: cloudify.gcp.nodes.Instance
  properties:
    ...
  relationships:
    - type: cloudify.gcp.relationships.contained_in_network
      target: my_subnet

{{< /gsHighlight >}}

If you want to use an exsisting SubNetwork (`use_external_resource: true`) then you must supply the `name` and `region` properties. This is because SubNetwork names are not unique across the whole project, only within a region.




## cloudify.gcp.nodes.TargetProxy
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

A TargetHttpProxy or TargetHttpsProxy.

Specify which using the `target_proxy_type` property.



**Properties:**


  * `ssl_certificate`
    URL of a SSL certificate associated with this target proxy. Can and must be used only with https type proxy.

    *type:* string
    *default:* 
  * `additional_settings`
    Additional setting for target proxy

    *default:* {}
  * `name`
    Optional target proxy name. By default it will be target proxy id.

    *type:* string
    *default:* 
  * `target_proxy_type`
    This field indicates if this target proxy is a HTTP or HTTPS based target proxy. Possible values are: 'http' and 'https'.

    *type:* string
    *default:* http
  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `url_map`
    URL of a URL map which specifies how traffic from this target proxy should be redirected.

    *type:* string
    *default:* 
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true) or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False





## cloudify.gcp.nodes.UrlMap
**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

Maps URLs to BackendServices



**Properties:**


  * `default_service`
    URL of a backend service to which this URL map will redirect traffic by default.

    *type:* string
    *default:* 
  * `gcp_config`
    A dictionary of values to pass to authenticate with the GCP API.

    *default:* {}
  * `additional_settings`
    Additional setting for url map

    *default:* {}
  * `use_external_resource`
    Indicate whether the resource exists and use existing (true) or if Cloudify should create new resource (false).

    *type:* boolean
    *default:* False
  * `name`
    Optional health check name. By default it will be URL map id.

    *type:* string
    *default:* 





## cloudify.gcp.nodes.Volume
**Derived From:** [cloudify.nodes.Volume]({{< relref "blueprints/built-in-types.md" >}})

A GCP Volume.

A virtual disk which can be attached to Instances.



**Properties:**


  * `additional_settings`
    Additional setting for volume

    *default:* {}
  * `name`
    Optional disk name. By default it will be disk id.

    *type:* string
    *default:* 
  * `gcp_config`
    A dictionary of values to pass to authenticate with the Google Cloud Platform API.

    *default:* {}
  * `image`
    The image of the Volume.

    *default:* 
  * `use_external_resource`
    Indicate whether the resource exists or if Cloudify should create the resource.

    *type:* boolean
    *default:* False
  * `size`
    Size of the Volume in GB.

    *type:* integer
    *default:* 10







# Relationships

## cloudify.gcp.relationships.contained_in_compute
**Derived From:** cloudify.relationships.contained_in


## cloudify.gcp.relationships.contained_in_network
**Derived From:** cloudify.relationships.contained_in


## cloudify.gcp.relationships.dns_record_connected_to_instance
**Derived From:** cloudify.relationships.connected_to


## cloudify.gcp.relationships.dns_record_connected_to_ip
**Derived From:** cloudify.relationships.connected_to


## cloudify.gcp.relationships.dns_record_contained_in_zone
**Derived From:** cloudify.relationships.contained_in


## cloudify.gcp.relationships.file_system_contained_in_compute
**Derived From:** cloudify.relationships.contained_in


## cloudify.gcp.relationships.forwarding_rule_connected_to_target_proxy
**Derived From:** cloudify.relationships.connected_to


## cloudify.gcp.relationships.instance_connected_to_disk
**Derived From:** cloudify.relationships.connected_to


## cloudify.gcp.relationships.instance_connected_to_instance_group
**Derived From:** cloudify.relationships.connected_to


## cloudify.gcp.relationships.instance_connected_to_ip
**Derived From:** cloudify.relationships.connected_to


## cloudify.gcp.relationships.instance_connected_to_keypair
**Derived From:** cloudify.relationships.connected_to


## cloudify.gcp.relationships.instance_connected_to_security_group
**Derived From:** cloudify.relationships.connected_to


## cloudify.gcp.relationships.instance_contained_in_network
**Derived From:** cloudify.relationships.contained_in


## cloudify.gcp.relationships.uses_as_backend
**Derived From:** cloudify.relationships.connected_to




# Account Information

The plugin needs access to your GCP auth credentials (via the [`gcp_config`](#common-properties) parameter) in order to operate (but see below about use within a manager).


# Using a Manager

## Bootstrapping
Use the [simple manager blueprint]{{< relref "/manager/bootstrapping" >}} to bootstrap a manager on a CentOS 7 Instance with at least 4GB of RAM.

## `gcp_config`
If you don't want to provide the `gcp_config` dictionary to every node in your blueprints, you can provide it, as `json`, at `/etc/cloudify/gcp_plugin/gcp_config`


## Networks

Instances in GCP are not able to communicate internally with instances in a different network.
This means that if you want to run Cloudify agents on your nodes they must be in the same network as the manager.

Additionally, a given network must choose either auto-subnets or manual subnets operation when created.
For maximum flexibility, `auto_subnets: false` is recommended, though this requires that subnets are created for any region you wish to place Instances in.

