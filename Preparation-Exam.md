https://justinoconnorcodes.files.wordpress.com/2021/09/terraform-cheatsheet-1.pdf
https://res.cloudinary.com/acloud-guru/image/fetch/c_thumb,f_auto,q_auto/https://acg-wordpress-content-production.s3.us-west-2.amazonaws.com/app/uploads/2020/11/terraform-cheatsheet-from-ACG.pdf

# Introduction
- _Infrastructure as Code_ is the process of managing infrastructure in a file or files rather than manually configuring resources in a user interface. A resource in this instance is any piece of infrastructure in a given environment, such as a virtual machine, security group, network interface, etc.

- _Terraform_ is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.

- _Configuration files_ describe to Terraform the components needed to run a single application or your entire datacenter. Terraform generates an execution plan describing what it will do to reach the desired state, and then executes it to build the described infrastructure. As the configuration changes, Terraform is able to determine what changed and create incremental execution plans which can be applied.

The infrastructure Terraform can manage includes low-level components such as compute instances, storage, and networking, as well as high-level components such as DNS entries, SaaS features, etc.

## Terraform's main features
<img src="https://miro.medium.com/max/1400/1*ozyZq7fXo1t10hvukS8KtQ.png" width="500"/>[^1]

## How it works
Terraform creates and manages resources on cloud platforms and other services through their APIs. Providers enable Terraform to work with virtually any platform or service with an accessible API.
<img src="https://miro.medium.com/max/1400/1*A1PWiPFasWKNePCgL6N_Cg.png" width="900"/>[^1]

## Configuration language
The main purpose of the Terraform language is declaring resources, which represent infrastructure objects. All other language features exist only to make the definition of resources more flexible and convenient.

A Terraform configuration is a complete document in the Terraform language (Hashicorp Configuration Language, HCL) that tells Terraform how to manage a given collection of infrastructure. A configuration can consist of multiple files and directories.

The syntax of the Terraform language consists of only a few basic elements:
- **Blocks:** containers for other content and usually represent the configuration of some kind of object, like a resource. Blocks have a block type, can have zero or more labels, and have a body that contains any number of arguments and nested blocks. Most of Terraform's features are controlled by top-level blocks in a configuration file.
- **Arguments:** assign a value to a name. They appear within blocks.
- **Expressions:** represent a value, either literally or by referencing and combining other values. They appear as values for arguments, or within other expressions.

These elements are structured as follows:
```
<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

Example:
```
resource "aws_vpc" "main" {
  cidr_block = var.base_cidr_block
}
```

The Terraform language is _declarative_, describing an intended goal rather than the steps to reach that goal. The ordering of blocks and the files they are organized into are generally not significant; Terraform only considers _implicit_ and _explicit_ relationships between resources when determining an order of operations.

# Expressions
[Expressions](https://www.terraform.io/docs/configuration/expressions) are used to refer to or compute values within a configuration. The simplest expressions are just literal values, like `"hello"` or `5`, but the Terraform language also allows more complex expressions such as references to data exported by resources, arithmetic, conditional evaluation, and a number of built-in functions.

The result of an expression is a value. All values have a [type](https://www.terraform.io/language/expressions/types#types=), which dictates where that value can be used and what transformations can be applied to it.

<img src="https://miro.medium.com/max/1400/1*RgNuNbnxCekhoIu-PgG-1Q.png" width="900"/>[^1]

The Terraform language uses the following types for its values:
- **Primitive types:** `string`, `number` and `boolean`. A primitive type is a simple type that isn't made from any other types. All primitive types in Terraform are represented by a type keyword. 
- **Complex types:** A complex type is a type that groups multiple values into a single value. Complex types are represented by type constructors, but several of them also have shorthand keyword versions. There are two categories of complex types: collection types (for grouping similar values), and structural types (for grouping potentially dissimilar values).
	- _Collection types:_ `list` and `map`. A collection type allows multiple values of one other type to be grouped together as a single value. The type of value within a collection is called its element type. All collection types must have an element type, which is provided as the argument to their constructor.
	- _Structural types:_ `tuple` and `object`. A structural type allows multiple values of several distinct types to be grouped together as a single value. Structural types require a schema as an argument, to specify which types are allowed for which elements.

We can define a `default` value for these types, for example:
```
variable "vpcname" {
    type = string
    default = "myvpc"
}
```

```
variable "enabled" {
    default = true
}
```

```
variable "sshport" {
    type = number
    default = 22
}
```

> **Note** 
> A `number` does not need double quotes, Terraform automatically converts `number` and `boolean` values to strings when needed. For example 5 and "5" both are correct.

## Complex type: List
`list` is the same as an array, we can store multiple values. 

Example:
```
variable "mylist" {
    type = list(string)
    default = ["Value1", "Value2"]
}
```
> **Note**
> Remember the first value is the 0 position. For example, the 0 position can be accessed through the following command: `var.mylist[0]`.

## Complex type: Map
`map` is a `<Key> = <Value>` pair. We use the key to access to the value. 

```
variable "mymap" {
    type = map
    default = {
        Key1 = "Value1"
        Key2 = "Value2"
    }
}
```

For example, if we need to access the value of Key1 (Value1) we can use the the following command: `var.mymap["Key1"]`.

> **Note** 
> Remember, we use `[ ]` for `list`, and we use `{ }` for `map`.

## Complex type: Tuple
The difference between a `tuple` and a `list`, is that for `list` we need to specify one dinstinct data type (`string` or `number`), whereas using `tuple` we are able to use multiple data types.

Example:
```
variable "mytuple" {
    type = tuple([string, number, string])
    default = ["cat", 1, "dog"]
}
```

## Complex type: Object
Similarly, using `object` we can use multiple data types instead of a specified one for `map`.

Example:
```
variable "myobject" {
    type = object({name = string, port = list(number)})
    default = {
        name = "TJ"
        port = [22, 25, 80]
    }
}
```

> **Note** 
> Using `object` and `tuple` allows us to have multiple values of several distinct types to be grouped as a single value.

# Blocks
A block is a container for other content.

Example:
```
resource "aws_instance" "example" {
  ami = "abc123"

  network_interface {
    # ...
  }
}
```

A block has a type (`resource` in this example). Each block type defines how many labels must follow the type keyword. The resource block type expects two labels, which are `aws_instance` and `example` in the example above. A particular block type may have any number of required labels, or it may require none as with the nested `network_interface` block type.

After the block type keyword and any labels, the block body is delimited by the `{ }` characters. Within the block body, further arguments and blocks may be nested, creating a hierarchy of blocks and their associated arguments.

The Terraform language uses a limited number of top-level block types, which are blocks that can appear outside of any other block in a configuration file. Most of Terraform's features (including resources, input variables, output values, data sources, etc.) are implemented as top-level blocks.

<img src="https://miro.medium.com/max/1400/1*b8enCRGjvJkO6texZBnEfg.png" width="600"/>[^1]

## (Input) Variables & Values
The Terraform language includes a few kinds of blocks for requesting or publishing named values.

- **Input variables:** serve as parameters for a Terraform module, so users can customize behavior without editing the source.
- **Output values:** are like return values for a Terraform module.
- **Local values:** are a convenience feature for assigning a short name to an expression.

### (Input) Variables
Each input variable accepted by a module must be declared using a variable block.  Terraform CLI defines optional arguments for variable declarations.

<img src="https://miro.medium.com/max/1400/1*_FWwGch6_ettk6ZvYPYwAw.png" width="800"/>[^1]

Input variables enables the end-user to set a variable manually after running `terraform plan`, we can add a `description` and after Terraform has run the plan it will show a message to enter a specific value.

Example:
```
variable "vpc_name" {
    type = string
    description = "Set VPC name"
}
```

The `terraform plan` command will prompt the following:

```
var.inputname
	Set VPC name
	Enter a value:
```

#### Assigning values to root module variables
When variables are declared in the root module of your configuration, they can be set in a number of ways: (1) in a Terraform Cloud workspace, (2) individually, with the `-var` command line option, (3) in variable definitions (`.tfvars`) files, (3a) either specified on the command line or (3b) automatically loaded or (4) as environment variables.

<img src="https://miro.medium.com/max/1400/1*a1XXIztHa2Et_g-pSftDSw.png" width="800"/>[^1]


##### The `-var` command line option
To specify individual variables on the command line, use the -var option when running the `terraform plan` and `terraform apply` commands, for example: `terraform plan -var="vpcname=cliname"`.

##### Variable definitions (`.tfvars`) files: via command line
To set lots of variables, it is more convenient to specify their values in a variable definitions file (with a filename ending in either `.tfvars` or `.tfvars.json`) and then specify that file on the command line with `-var-file`:

Example:
```
vpcname = "tfvarsname"
port = 22
policy = {
	test = 1
	debug = "true"
}
```

```
terraform apply -var-file="testing.tfvars"
```

> **Note** 
> This is how Terraform Cloud passes workspace variables to Terraform.

> **Warning** 
> The `terraform.tfvars` file is used to define variables whereas the `.tf` file declares that the variable exists.[^3]

[^3]: <https://amazicworld.com/difference-between-variable-tf-and-variable-tfvars-in-terraform>

##### Variable definitions (`.tfvars`) files: automatically loaded
Terraform also automatically loads a number of variable definitions files if they are present. Files named exactly `terraform.tfvars` or `terraform.tfvars.json`. Also, any files with names ending in `*.auto.tfvars` or `*.auto.tfvars.json`, this is useful for setting variables for different environments.

##### Environment variables
As a fallback for the other ways of defining variables, Terraform searches the environment of its own process for environment variables named `TF_VAR_...` followed by the name of a declared variable. We can create an export with our variable before execute `terraform plan`, and overwrite the value on the `.tf` files, for example `export TF_VAR_vpcname=envvpc`. This is useful to pass secrets or sensitive information in a secure way.

##### Variable definition precedence
Terraform loads variables in the following order, with **later** sources taking precedence over earlier ones:
- Environment variables
- The `terraform.tfvars` file or the `terraform.tfvars.json` file, if present.
- Any `*.auto.tfvars` or `*.auto.tfvars.json` files, processed in lexical order of their filenames.
- Any `-var` and `-var-file` options on the command line, in the order they are provided (this includes variables set by a Terraform Cloud workspace).

> **Note** 
> There is no mention of `.tf` file declaration here, this is because variables declared in `.tf` files are concatenated into a single entity consisting of your `variables.tf` your `main.tf` and your `output.tf` files before being processed by Terraform. Hence, this declaration has the highest precedence.

### Ouput values (outputs)
Output values make information about your infrastructure available on the command line, and can expose information for other Terraform configurations to use. Output values are similar to return values in programming languages.

> **Note**
> For brevity, output values are often referred to as just "outputs" when the meaning is clear from contex. More importantly, oputputs are only rendered when Terraform applies your plan. Running `terraform plan` will not render outputs.

```
output "vpc_id" {
    value = "aws_vpc.myvpc.id"
}
```

If we run `terraform apply`, we will see the next message printed:

```
Apply complete!
Outputs:
vpcid = vpc-099d9099f5faec2d9
```

### Local values (locals)

A local value assigns a name to an [expression](https://www.terraform.io/docs/configuration/expressions), allowing it to be used multiple times within a module without repeating it.

Comparing modules to functions in a traditional programming language: if [input variables](https://www.terraform.io/docs/configuration/variables) are analogous to function arguments and [outputs values](https://www.terraform.io/docs/configuration/outputs) are analogous to function return values, then *local values* are comparable to a function's local temporary symbols.

> **Note** 
> For brevity, local values are often referred to as just "locals" when the meaning is clear from context.

**Declaring a local value:**
A set of related local values can be declared together in a single `locals` block.

```
locals {
  service_name = "forum"
  owner        = "Community Team"
}
```

The _expressions_ assigned to local value names can either be simple constants like the above, allowing these values to be defined only once but used many times, or they can be more complex expressions that transform or combine values from elsewhere in the module:

**When to use local values:**
Local values can be helpful to avoid repeating the same values or expressions multiple times in a configuration, but if overused they can also make a configuration hard to read by future maintainers by hiding the actual values used.

Use local values only in moderation, in situations where a single value or result is used in many places *and* that value is likely to be changed in future. The ability to easily change the value in a central place is the key advantage of local values.

## Resources
Resources are the most important element in the Terraform language. Each resource block describes one or more infrastructure objects, such as virtual networks, compute instances, or higher-level components such as DNS records. For example:

```
resource "aws_instance" "web" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
}
```

The resource block above declares a resource of a given type (`"aws_instance"`) with a given local name ("web"). The name is used to refer to this resource from elsewhere in the same Terraform module, but has no significance outside that module's scope.

The resource type and name together serve as an identifier for a given resource and so must be unique within a module.

Within the block body (between `{` and `}`) are the configuration arguments for the resource itself. Most arguments in this section depend on the resource type, and indeed in this example both ami and `instance_type` are arguments defined specifically for the `aws_instance` resource type.

> **Note** 
> Resource names must start with a letter or underscore, and may contain only letters, digits, underscores, and dashes.

### Meta-Arguments
The Terraform language defines several meta-arguments, which can be used with any resource type to change the behavior of resources.

The following meta-arguments are documented on separate pages:

- `depends_on`, for specifying hidden dependencies
- `count`, for creating multiple resource instances according to a count
- `for_each`, to create multiple instances according to a map, or set of strings
- `provider`, for selecting a non-default provider configuration
- `lifecycle`, for lifecycle customizations
- `provisioner`, for taking extra actions after resource creation

#### Dependencies

Explicitly specifying a dependency is only necessary when a resource relies on some other resource's behavior but *doesn't* access any of that resource's data in its arguments.

```
resource "aws_instance" "example" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
  depends_on = [aws_iam_role_policy.example]
}
```

The `depends_on` meta-argument, if present, must be a list of references to other resources in the same module.

Link: <https://www.terraform.io/docs/providers/aws/d/instance>

#### Provisioners

_Provisioners_ can be used to model specific actions on the local machine or on a remote machine in order to prepare servers or other infrastructure objects for service.

> **Note** 
> Provisioners should only be used as a last resort. For most common situations there are better alternatives.

Example:
```
resource "aws_instance" "web" {
  # ...
  provisioner "local-exec" {
    command = "echo The server's IP address is ${self.private_ip}"
  }
}
```

The `local-exec` provisioner requires no other configuration, but most other provisioners must connect to the remote system using SSH or WinRM.

##### Creation-time provisioners
By default, provisioners run when the resource they are defined within is created. Creation-time provisioners are only run during *creation*, not during updating or any other lifecycle. They are meant as a means to perform bootstrapping of a system. If a creation-time provisioner fails, the resource is marked as tainted. A tainted resource will be planned for destruction and recreation upon the next `terraform apply`

##### Destroy-time provisioners
If `when = "destroy"` is specified, the provisioner will run when the resource it is defined within is *destroyed*.

```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    when    = "destroy"
    command = "echo 'Destroy-time provisioner'"
  }
}
```

Destroy provisioners are run before the resource is destroyed. If they fail, Terraform will error and rerun the provisioners again on the next `terraform apply`

> **Note** 
> By default, a defined provisioner is a creation-time provisioner. You must explicitly define a provisioner to be a destroy-time provisioner

##### Local-Exec vs. Remote-Exec
With Terraform the plugins have two options to do the job:
-   [**Local-Exec:**](https://www.terraform.io/docs/provisioners/local-exec) From our local machine
-   [**Remote-Exec:**](https://www.terraform.io/docs/provisioners/remote-exec) On the remote instance

One example of local-exec is create a ssh key in our machine.

```
resource "null_resource" "generate-sshkey" {
    provisioner "local-exec" {
        command = "yes y | ssh-keygen -b 4096 -t rsa -C 'terraform-vsphere-kubernetes' -N '' -f ${var.virtual_machine_kubernetes_controller.["private_key"]}"
    }
}
```

Another example for `local-exec` is executing  a script for download AWS  Lambda dependencies, and after that, make a zip.

One example for `remote-exec` is from the key create previously, we can configure on a deployed virtual machine

```
  provisioner "remote-exec" {
    inline = [
      "mv /tmp/authorized_keys /root/.ssh/authorized_keys",
      "chmod 600 /root/.ssh/authorized_keys",
    ]
    connection {
      type          = "${var.virtual_machine_template.["connection_type"]}"
    }
  }
```

## Providers
Providers are plugins that implement resource types. A provider is responsible for understanding API interactions and exposing resources. If an API is available, you can create a provider. A provider uses a plugin. In order to make a provider available on Terraform, we need to run a `terraform init`, this command download any plugins we need for our providers. If for example we need to copy the plugin directory manually, we can do it, moving the files to `.terraform.d/plugins`.

> **Note** 
> Using the `terraform providers` command we can view the specified version constraints for all providers used in the current configuration.

<img src="https://miro.medium.com/max/1400/1*Vyb5RNl3PhxytQsD0vaH-w.png" width="800"/>[^1]

Example configuration:
```
terraform {
  required_providers {
    aws = "~> 2.7"
  }
}
```

Check:
```
$ terraform providers
.
└── provider.aws ~> 2.7
```

When `terraform init` is re-run with providers already installed, it will use an already-installed provider that meets the constraints in preference to downloading a new version. To upgrade to the latest acceptable version of each provider, run `terraform init -upgrade`. This command also upgrades to the latest versions of all Terraform modules.

### Multi-provider set-up

We can use for example multiple AWS providers with different regions, for this we need to create an `alias` and on the resource creation we need to specified the provider. For example

```
provider "aws" {
	region = "us-east-1"
}

provider "aws" {
	region = "us-west-1"
	alias = "ireland"
}

resource "aws_vpc" "irlvpc" {
	cidr_block = "10.0.0.0/16"
	provider   = "aws.ireland"
}
```

## Data sources
Data sources allow Terraform to use information defined outside of Terraform, defined by another separate Terraform configuration, or modified by functions. Through data sources, Terraform can query AWS and return results (API request to get information). For example we can retrieve information of an EC2 serveron AWS (even without being created through Terraform).

A data source is accessed via a special kind of resource known as a `data` resource, declared using a `data` block.

Example of using a data source to retrieve the AZ of an EC2, the `output` block will print the AZ:

```
# Find the latest available AMI that is tagged with Component = "DB server"
data "aws_instance" "dbsearch" {
filter {
	name = "tag:Name"
	values = ["DB server"]
 }
}
output "dbserver" {
 value = data.aws_instance.dbsearch.availability_zone
}
```
A data block request that Terraform read from a given data source and export the result under the give local name.

> **Note**
> Data source attributes are interpolated with the general syntax `*data.TYPE.NAME.ATTRIBUTE*`. The interpolation for a resource is the same but without the` *data.*` prefix (`TYPE.NAME.ATTRIBUTE`).

## Terraform Settings
<img src="https://miro.medium.com/max/1400/1*3nnDHIS3zQ2W0CsGCjsHvQ.png" width="800"/>[^1]

### Versioning
[The `required_version` setting](https://www.terraform.io/docs/configuration/terraform#specifying-a-required-terraform-version) can be used to constrain which versions of the Terraform CLI can be used with your configuration. If the running version of Terraform doesn't match the constraints specified, Terraform will produce an error and exit without taking any further actions.

```
terraform {
  required_version = ">= 0.12"
}
```

The value for `required_version` is a string containing a comma-separated list of constraints. Each constraint is an operator followed by a version number, such as `> 0.12.0`. The following constraint operators are allowed:

-   `=` (or no operator): exact version equality
-   `!=`: version not equal
-   `>`, `>=`, `<`, `<=`: version comparison, where "greater than" is a larger version number
-   `~>`: _pessimistic_ constraint operator, constraining both the oldest and newest version allowed. For example, `~> 0.9` is equivalent to `>= 0.9, < 1.0`, and `~> 0.8.4`, is equivalent to `>= 0.8.4, < 0.9`

We can also specified a provider version requirement:
```
provider "aws" {
	region = "us-east-1"
	version = ">= 2.9.0"
}
```

## Dynamic blocks
Within top-level block constructs like resources, expressions can usually be used only when assigning a value to an argument using the `name = expression` form. This covers many uses, but some resource types include repeatable *nested blocks* in their arguments.

[You can dynamically construct repeatable nested blocks](https://www.terraform.io/language/expressions/dynamic-blocks) like `setting` using a special `dynamic` block type, which is supported inside `resource`, `data`, `provider`, and `provisioner` blocks:

```
resource "aws_elastic_beanstalk_environment" "tfenvtest" {
  name                = "tf-test-name"
  application         = "${aws_elastic_beanstalk_application.tftest.name}"
  solution_stack_name = "64bit Amazon Linux 2018.03 v2.11.4 running Go 1.12.6"

  dynamic "setting" {
    for_each = var.settings
    content {
      namespace = setting.value["namespace"]
      name = setting.value["name"]
      value = setting.value["value"]
    }
  }
}
```

A `dynamic` block acts much like a `for` expression, but produces nested blocks instead of a complex typed value. It iterates over a given complex value, and generates a nested block for each element of that complex value.

## Modules
Modules are self-contained packages of Terraform configurations that are managed as a group. A module is a simple directory that contains other `.tf` files. Using modules we can make our code re-usable. Modules are either local or remote.

<img src="https://miro.medium.com/max/1400/1*ItQg-iUT0O3QDiLoJBndJg.png" width="600"/>[^2]
<img src="https://miro.medium.com/max/1400/1*ilau3dR50ZfKadoc1_TO_w.png" width="600"/>[^2]

Modules can be found on [Terraform Registry](https://registry.terraform.io/browse/modules), theses modules are verified by HashiCorp

### Modules inputs/outputs
For make modules inputs we use inputs variables. 

Example module code:
```
variable "dbname" {
	type = string
}
resource "aws_instance" "myec2db" {
	ami = "ami-01a6e"
	tags = {
		Name = var.dbname
	}
}
```

Call to the module example:
```
module "dbserver" {
	source = "./db"
	dbname = "mydbserver"
}
```

Module outputs are very similar to module inputs, an example in a module output:
```
output "privateip" {
	value = aws_instance.myec2db.private_ip
}
```

When we use a module with an output, to use the output we need to specified in the call to our module, for example:
```
module "dbserver" {
	source = "./db"
	dbname = "mydbserver"
}

output "dbprivateip" {
	value = module.dbserver.privateip
}
```
<img src="https://miro.medium.com/max/1400/1*nC50N58BRmDPkIKGHQOqbA.png" width="800"/>[^2]

### Child modules
Terraform allows for _child modules_, modules within modules, so to say. This basically is a directory with `.tf` files, within other directories with others sub modules. After creating a subdirectory, remember to re-run the `terraform init` command.

<img src="https://miro.medium.com/max/1400/1*7OER_lXpP-25B1LKW1lPOw.png" width="800"/>[^2]

### Module compositions
The key features of modules are _re-usability_ and _composability_. Below are some patterns to keep in mind when creating flexible, re-usable and composable modules.

> Keep a flat tree of module calls. Try to avoid having children modules that have their own children: The [Terraform documentation](https://www.terraform.io/language/modules/develop/composition#module-composition) recommends only one level of child modules.

<img src="https://miro.medium.com/max/1400/1*N6YMOp-V3uQcF_sHFoztNA.png" width="700"/>[^2]

> Keep modules relatively small and pass in dependencies instead (_dependency inversion_).

<img src="https://miro.medium.com/max/1400/1*VOG5X8xSEiFCoAFenQ66hQ.png" width="700"/>[^2]

In the example above the network creation is separated from the redis module. This makes it easy for the resources defined by the module to coexist with other infrastructure in the same network.

> Avoid complex conditional branches when creating objects within modules. Using an input variable is more declarative.

<img src="https://miro.medium.com/max/1400/1*expkYWhUbQfQLsOTYVXQxQ.png" width="800"/>[^2]

# Commands

## The core workflow
The core Terraform workflow consists of three stages:
1. **Write:** You define resources, which may be across multiple cloud providers and services. For example, you might create a configuration to deploy an application on virtual machines in a Virtual Private Cloud (VPC) network with security groups and a load balancer.
2. **Plan:** Terraform creates an execution plan describing the infrastructure it will create, update, or destroy based on the existing infrastructure and your configuration.
3. **Apply:** On approval, Terraform performs the proposed operations in the correct order, respecting any resource dependencies. For example, if you update the properties of a VPC and change the number of virtual machines in that VPC, Terraform will recreate the VPC before scaling the virtual machines.

<img src="https://miro.medium.com/max/1400/1*E6p3Q7PGrlPtLSxaYxBtAQ.png" width="600"/>[^1]

There are three types of users, the workflow changes based on the user type:
| Stage / User type | **Individual** | **Team** | **Terraform Cloud** |
| ----- | ----- | ----- | ----- |
| Write | Create the Terraform files | Create the Terraform files and Checkout the latest code | Use Terraform Cloud as your `development` environment (statefiles, variables and secrets on Terrafom Cloud) |
| Plan | Run Terraform plan and check | Run Terraform Plan and raise a Pull-Request | When a PR is raised, Terraform Plan is run |
| Create | Create the infrastructure | Merge and create | Before merging a second plan is run before approval to create |

### terraform init
[The `terraform init` command](https://www.terraform.io/docs/commands/init.html) is used to initialize a working directory containing Terraform configuration files. It is safe to run this command multiple times, , this command will never delete your existing configuration or state. During init, the root configuration directory is consulted for [backend configuration](https://www.terraform.io/docs/backends/config.html) and the chosen backend is initialized using the given configuration settings.

<img src="https://miro.medium.com/max/1400/1*z6bsYznAVDzqvpfL3xxbXw.png" width="800"/>[^1]

### terraform validate
[The `terraform validate` command](https://www.terraform.io/docs/commands/validate.html) validates the configuration files in a directory, referring only to the configuration and not accessing any remote services such as remote state, provider APIs, etc.

Validate runs checks that verify whether a configuration is syntactically valid and internally consistent, regardless of any provided variables or existing state.

It is thus primarily useful for general verification of reusable modules, including correctness of attribute names and value types.

It is safe to run this command automatically, for example as a post-save check in a text editor or as a test step for a re-usable module in a CI system.

> **Note** 
> Validation requires an initialized working directory with any referenced plugins and modules installed.

### terraform plan
[The `terraform plan` command](https://www.terraform.io/docs/commands/plan.html) is used to create an execution plan. Terraform performs a refresh, unless explicitly disabled, and then determines what actions are necessary to achieve the desired state specified in the configuration files.

<img src="https://miro.medium.com/max/1400/1*3Y1M38zlOEnurCxBHbMg9w.png" width="600"/>[^1]

### terraform apply
[The `terraform apply` command](https://www.terraform.io/docs/commands/apply.html) is used to apply the changes required to reach the desired state of the configuration, or the pre-determined set of actions generated by a `terraform plan` execution plan.

<img src="https://miro.medium.com/max/1400/1*SSXJh0WapTdixVEg0GDVsA.png" width="800"/>[^1]

### terraform destroy
[The `terraform destroy` command](https://www.terraform.io/docs/commands/destroy.html) is used to destroy the Terraform-managed infrastructure.


## Other commands

### terraform fmt
The `terraform fmt` command is used to rewrite Terraform configuration files to a canonical format and style. The canonical format may change in minor ways between Terraform versions, so after upgrading Terraform it is recommended to proactively run `fmt`

### terraform taint
The `terraform taint` command manually marks a Terraform-managed resource as tainted, forcing it to be destroyed and recreated on the next apply. Taint force the recreation. This command *will not* modify infrastructure, but does modify the state file in order to mark a resource as tainted. Once a resource is marked as tainted, the next [plan](https://www.terraform.io/docs/commands/plan.html) will show that the resource will be destroyed and recreated and the next [apply](https://www.terraform.io/docs/commands/apply.html) will implement this change.

Forcing the recreation of a resource is useful when you want a certain side effect of recreation that is not visible in the attributes of a resource. For example: re-running provisioners will cause the node to be different or rebooting the machine from a base image will cause new startup scripts to run.

Example:

```
$ terraform taint aws_vpc.myvpc
The resource aws_vpc.myvpc in the module root has been marked as tainted.
```

Another example if we want taint the resource "aws_instance" "baz" resource that lives in module bar which lives in module foo.

```
terraform taint module.foo.module.bar.aws_instance.baz
```

Link: <https://www.terraform.io/docs/internals/resource-addressing.html>

### terraform untaint

The `terraform untaint` command manually unmark a Terraform-managed resource as tainted, restoring it as the primary instance in the state.

> **Note** 
> This command *will not* modify infrastructure, but does modify the state file in order to unmark a resource as tainted.

```
$ terraform untaint aws_vpc.myvpc
Resource aws_vpc.myvpc2 has been successfully untainted.
```

### terraform import
[The `terraform import` command](https://www.terraform.io/docs/commands/import.html>) is used to [import existing resources](https://www.terraform.io/docs/import/index.html) into Terraform.

Import will find the existing resource from ID and import it into your Terraform state at the given ADDRESS.

ADDRESS must be a valid [resource address](https://www.terraform.io/docs/internals/resource-addressing.html). Because any resource address is valid, the import command can import resources into modules as well directly into the root of your state.

ID is dependent on the resource type being imported. For example, for AWS instances it is the instance ID (`i-abcd1234`) but for AWS Route53 zones it is the zone ID (`Z12ABC4UGMOZ2N`).

**Usage:** `terraform import [options] ADDRESS ID`

Example:

```
$ terraform import aws_vpc.vpcimport vpc-06f0e46d612
```

> **Note**
>  `terraform import` command can import resources directly into modules

### terraform show

The `terraform show` command is used to provide human-readable output from a state or plan file. This can be used to inspect a plan to ensure that the planned operations are expected, or to inspect the current state as Terraform sees it.

Usage: `terraform show [options] [path]`

You may use `show` with a path to either a Terraform state file or plan file. If no path is specified, the current state will be shown.

### terraform workspace

The `terraform workspace` command is used to manage [workspaces](https://www.terraform.io/docs/state/workspaces.html). It is useful to split and separate the state files. This command is a container for further subcommands such as `list`, `select`, `new`, `delete` and `show`

Usage: `terraform workspace <subcommand> [options] [args]`

If we never create a workspace we use the default workspace

```
# terraform workspace list
* default
```

> **Note**
> For local state, Terraform stores the workspace states in a directory called `terraform.tfstate.d`. This directory should be treated similarly to local-only `terraform.tfstate`

> **Note**
> Terraform Cloud and Terraform CLI both have features called "workspaces," but they're slightly different. CLI workspaces are alternate state files in the same working directory; they're a convenience feature for using one configuration to manage multiple similar groups of resources.

#### terraform workspace new
The `terraform workspace new` command is used to create a new workspace.

Example:
```
$ terraform workspace new dev
Created and switched to workspace "dev"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
```

Now, if we execute a `terraform plan` in our environment, the plan will show many resources to create, because this is a different state file.

#### terraform workspace show

The `terraform workspace show` command is used to output the current workspace.

```
$ terraform workspace show
dev
```

#### terraform workspace select
The `terraform workspace select` command is used to choose a different workspace to use for further operations. First we need to execute a `terraform workspace list` to know the workspaces names.

```
$ terraform workspace select default
Switched to workspace "default".
```

#### terraform workspace delete
The `terraform workspace delete` command is used to delete an existing workspace.

```
$ terraform workspace delete dev
Deleted workspace "dev"!
```

## terraform state

### terraform state list
[The `terraform state list` command](https://www.terraform.io/docs/commands/state/list.html) is used to list resources within a [Terraform state](https://www.terraform.io/docs/state/index.html)

The command will list all resources in the state file matching the given addresses (if any). If no addresses are given, all resources are listed.

Example of listing all resources:

```
$ terraform state list
aws_instance.foo
aws_instance.bar[0]
aws_instance.bar[1]
module.elb.aws_elb.main
```

Example of filtering by resource:

```
$ terraform state list aws_instance.bar
aws_instance.bar[0]
aws_instance.bar[1]
```

### terraform state pull
The `terraform state pull` command is used to manually download and output the state from [remote state](https://www.terraform.io/docs/state/remote.html). This command also works with local state (but is not very useful because we can see the local file)

This command will download the state from its current location and output the raw format to stdout.

This is useful for reading values out of state (potentially pairing this command with something like [jq](https://stedolan.github.io/jq/)). It is also useful if you need to make manual modifications to state.

### terraform state mv
[The `terraform state mv` command](<https://www.terraform.io/docs/commands/state/mv.html) is used to move items in a [Terraform state](https://www.terraform.io/docs/state/index.html). This command can move single resources, single instances of a resource, entire modules, and more. This command can also move items to a completely different state file, enabling efficient refactoring.

Usage: `terraform state mv [options] SOURCE DESTINATION`

This can be used for simple resource renaming, moving items to and from a module, moving entire modules, and more. And because this command can also move data to a completely new state, it can also be used for refactoring one configuration into multiple separately managed Terraform configurations.

Example:
```
terraform state mv 'packet_device.worker' 'packet_device.helper'
```

### terraform state rm
[The `terraform state rm` command](<https://www.terraform.io/docs/commands/state/rm.html) is used to remove items from the [Terraform state](https://www.terraform.io/docs/state/index.html). This command can remove single resources, single instances of a resource, entire modules, and more.

Usage: `terraform state rm [options] ADDRESS...`.

Items removed from the Terraform state are *not physically destroyed*. Items removed from the Terraform state are only no longer managed by Terraform. For example, if you remove an AWS instance from the state, the AWS instance will continue running, but `terraform plan` will no longer see that instance.

There are various use cases for removing items from a Terraform state file. The most common is refactoring a configuration to no longer manage that resource (perhaps moving it to another Terraform configuration/state).

Example remove a resource:

```
$ terraform state rm 'packet_device.worker'
```

Example remove a module:

```
$ terraform state rm 'module.foo'
```

# Summary commands

Usage: `terraform [global options] <subcommand> [args]`

The available commands for execution are listed below. The primary workflow commands are given first, followed by less common or more advanced commands.

**Main commands:**
	
- `init`          Prepare your working directory for other commands
- `validate`      Check whether the configuration is valid
- `plan`          Show changes required by the current configuration
- `apply`         Create or update infrastructure
- `destroy`       Destroy previously-created infrastructure

**All other commands:**
	
- `console`       Try Terraform expressions at an interactive command prompt
- `fmt`           Reformat your configuration in the standard style
- `force-unlock`  Release a stuck lock on the current workspace
- `get`           Install or upgrade remote Terraform modules
- `graph`         Generate a Graphviz graph of the steps in an operation
- `import`        Associate existing infrastructure with a Terraform resource
- `login`         Obtain and save credentials for a remote host
- `logout`        Remove locally-stored credentials for a remote host
- `output`        Show output values from your root module
- `providers`     Show the providers required for this configuration
- `refresh`       Update the state to match remote systems
- `show`          Show the current state or a saved plan
- `state`         Advanced state management
- `taint`         Mark a resource instance as not fully functional
- `untaint`       Remove the 'tainted' state from a resource instance
- `version`       Show the current Terraform version
- `workspace`     Workspace management

**Global options (use these before the subcommand, if any):**
	
- `-chdir=DIR`    Switch to a different working directory before executing thegiven subcommand.
- `-help`         Show this help output, or the help for a specified subcommand.
- `-version`      An alias for the "version" subcommand.

# Built-in functions
The Terraform language includes a number of built-in functions that you can call from within expressions to transform and combine values. The general syntax for function calls is a function name followed by comma-separated arguments in parentheses:

```
max(5, 12, 9)

```

> **Note**
> The Terraform language does not support user-defined functions, and so only the functions built in to the language are available for use.

A very useful function is the `file` function, using that, we can read the contents of a file and returns them as a string. Another important function is the `flatten` function, this takes a list and replaces any elements that are list with a flattened sequence of the list contents.

The last important function is `lookup`, this retrieves the value of a single element from a map, given its key. If the given key does not exist, a the given default value is returned instead.

```
lookup(map, key, default)
```

Examples:

```
> lookup({a="ay", b="bee"}, "a", "what?")
ay
> lookup({a="ay", b="bee"}, "c", "what?")
what?
```

Links:

-   <https://www.terraform.io/docs/configuration/functions.html>
-   <https://www.terraform.io/docs/configuration/expressions.html#function-calls>


# State

## Local state
The local backend stores state on the local file system, locks that state using system APIs, and performs operations locally.

```
terraform {
  backend "local" {
    path = "relative/path/to/terraform.tfstate"
  }
}
```

## Partial configuration
You do not need to specify every required argument in the backend configuration. Omitting certain arguments may be desirable to avoid storing secrets, such as access keys, within the main configuration. When some or all of the arguments are omitted, we call this a *partial configuration*.

With a partial configuration, the remaining configuration arguments must be provided as part of [the initialization process](https://www.terraform.io/docs/backends/init.html#backend-initialization). There are several ways to supply the remaining arguments:

-   _Interactively:_ Terraform will interactively ask you for the required values
-   _File:_ A configuration file may be specified via the `init` command line. To specify a file, use the `-backend-config=PATH` option when running `terraform init`
-   _Command line key/value pairs:_ Key/value pairs can be specified via the `init` command line.

## terraform refresh
[The `terraform refresh` command](https://www.terraform.io/docs/commands/refresh.html) is used to reconcile the state Terraform knows about (via its state file) with the real-world infrastructure. This can be used to detect any drift from the last-known state, and to update the state file.

This does not modify infrastructure, but does modify the state file. If the state is changed, this may cause changes to occur during the next plan or apply.

Usage: `terraform refresh [options] [dir]`.

For example, if we change some configuration using the web console, we need to do:

```
$ terraform refresh
$ terraform plan

```

`refresh` is good practice to sync changes, in some cases it is better to use `terraform import` depending on the complexity.

## Remote backends
Stores the state as a given key in a given bucket on [Amazon S3](https://aws.amazon.com/s3/). This backend also supports state locking and consistency checking via [Dynamo DB](https://aws.amazon.com/dynamodb/), which can be enabled by setting the `dynamodb_table` field to an existing DynamoDB table name. A single DynamoDB table can be used to lock multiple remote state files. Terraform generates key names that include the values of the `bucket` and `key` variables.

Example:

```
terraform {
  backend "s3" {
    bucket = "mybucket"
    key    = "state/terraform.tfstate"
    region = "us-east-1"
  }
}
```

> **Note**
> This assumes we have a bucket created called `mybucket`. The Terraform state is written to the key `state/terraform.tfstate`.

> **Note**
> If we have a local state, and after that we change to the S3 backend, when we execute `terraform init` the system ask us if we want to copy existing state to the new backend.

Similar to if we delete a remote backend from the configuration, when we execute the re-initialization, Terraform will ask if we would like to migrate our state back down to normal local state.

Link: <https://www.terraform.io/docs/backends/types/s3.html>

## Revert to local backend
If we want to change from S3 backend to Local backend, only we need to do `terraform destroy` after that delete `backend.tf` file, and run `terraform init`

## Backend limitations & security
When we use Terraform is only allowed one backend. The state cannot store secrets, for that reason we need to encrypt at rest.

## terraform force-unlock
[The `terraform force-unlock` command](<https://www.terraform.io/docs/commands/force-unlock.html) will manually unlock the state for the defined configuration. This will not modify your infrastructure. This command removes the lock on the state for the current configuration. The behavior of this lock is dependent on the backend being used. Local state files cannot be unlocked by another process.

Usage: `terraform force-unlock LOCK_ID [DIR]`

This is useful for example when another person write the remote state file, and something happening and the state file is lock, and we need to release.

## Terraform Cloud
If we need the best solution for encryption and backups, the answer is Terraform Cloud. Terraform Cloud encrypt the state file at rest, and also encrypt using TLS.

## State in memory
When we use remote states, Terraform save all the temporal information in memory, nothing persistent on disk, this is another Terraform security implementation.

## Backend & string interpolation
Backend cannot have any interpolations or use any variables. This need to be manually hardcoded.

## terraform state push
The `terraform state push` command is used to manually upload a local state file to [remote state](https://www.terraform.io/docs/state/remote.html). This command also works with local state.

Usage: `terraform state push [options] PATH`

Link: <https://www.terraform.io/docs/commands/state/push.html>

## Types of backend
-   **Standard:** state management, storage and locking
-   **Enhanced:** Only on Terraform Cloud, standard + can run operations remotely

> **Note**
> Backends that support state lockings: (1) AzureRM, (2) Consul, (3) AWS S3

> **Note**
> - Only _one_ backend allowed
> - Secrets are stored in state
> - State locking when working with teams
> - State is stored in memory when using a remote backend
> - _Standard_ and _Enhanced_ backends
> - No interpolation allowed in backend setup
> - `terraform refresh` will attempt to re-sync the state
> - `terraform state push` will override the state

# Terraform Cloud & Enterprise

## Terraform Cloud
Terraform Cloud (TFC) is a free to use, self-service SaaS platform that extends the capabilities of the open source Terraform CLI and adds collaboration and automation features.

Terraform Cloud enables connecting to common VCS platforms (GitHub, GitLab, Bitbucket) and triggering Terraform runs (plan and apply) from changes to configuration within the VCS. TFC manages state for the user, including keeping a history of changes. Terraform Cloud exposes an HTTP API that anyone can integrate with to build more automation around infrastructure change.

## Terraform Cloud set-up
First we need to create an account on <https://app.terraform.io/signup/account> and create a new Git repository, and upload our `.tf` files.

After that on Terraform Cloud web page we need to create an Organization, only we need to specified the name, and one email address, after that we can connect to our Git repository (GitHub, GitLab, Bitbucket and Azure DevOps are available), also we need to authorize Terraform Cloud in our Git repository, is only one click.

Also, we can implement a _Private module registry_ in Terraform Cloud.

Now we need to create a Workspace on Terraform Cloud (is different to the Workspace on Terraform), this is for create a separate environment, for example dev, prod, etc.

Right now we need to Configure Variables, because we used AWS provider, we need to configure the AWS credentials (Also we can configure using Vault)

Lastly, we need to configure a _Queue Plan_, this allow us to execute a `terraform plan` after this complete, we can Confirm & Apply

> **Note** The remote backend stores the Terraform state and may be used to run operations in Terraform Cloud. When using full remote operations, operations like `terraform plan` or `terraform apply` can be executed in Terraform Cloud's run environment, with log output streaming to the local terminal.

> **Note** Workspaces, managed with the `terraform workspace` command, are not the same thing as Terraform Cloud's workspaces. Terraform Cloud workspaces act more like completely separate working directories; CLI workspaces are just alternate state files.

> **Note** Terraform Cloud always encrypts state at rest and protects it with TLS in transit.

If we want to do a Destroy, we only need to click on the option "Queue destroy plan"

## OSS vs. Terraform Cloud

| Component | OSS | Terraform Cloud |
| --- | --- | --- |
| Terraform Configuration | On Disk | In linked version control repository, or periodically updated via API/CLI |
| Variable values | As .tfvars file, as CLI arguments, or in shell environment | In workspace |
| State | On disk or in remote backend | In workspace |
| Credential and secrets | In shell environment or entered at prompts | In workspace, stored as sensitive variables |

## Terraform Cloud: Paid features

Some paid features are Roles & Team management, Cost Estimation and Sentinel.

Up to five users we can have the free tier, with VCS integration, Workspace Management, Secure Variable Storage, Remote Runs & Applies, Full API Coverage and Private Module Registry.

Link: <https://www.hashicorp.com/products/terraform/pricing/>

## Terraform Cloud vs. Terraform Enterprise

Terraform Enterprise is a hosted version of Terraform Cloud, but using Terraform Enterprise we can have the following features:

-   SAML/SSO

-   Audit Logs

-   Private Network Connectivity

-   Clustering

Sentinel, VCS Integration are offered also in Terraform Cloud. Everything that is in Terraform Cloud is already included in Terraform Enterprise.

# Appendix

## Securing keys
In the case of using AWS provider the best practice for store credentials is having the keys in the AWS config file `.aws/credentials`, and not having the credentials anywhere in the Terraform code.

## Sentinel
[Sentinel](https://www.hashicorp.com/sentinel) is an embedded policy-as-code framework integrated with the HashiCorp Enterprise products. It enables fine-grained, logic-based policy decisions, and can be extended to use information from external sources.

## Secret injection
Vault allows you to manage secrets and protect sensitive data. Secure, store and tightly control access to tokens, passwords, certificates, encryption keys for protecting secrets and other sensitive data using a UI, CLI, or HTTP API.

## Files to be ignored by Git
The following Terraform files should be ignored by Git when committing code to a repo:
- The `terraform.tfstate` or `.auto.tfvars` files contains the terraform state of a specific environment and doesn't need to be preserved in a repo.
- The `terraform.tfvars` file may contain sensitive data, such as passwords or IP addresses of an environment that you may not want to share with others.

## Debugging in Terraform
[Terraform has detailed logs](https://www.terraform.io/docs/internals/debugging.html) which can be enabled by setting the `TF_LOG` environment variable to any value. This will cause detailed logs to appear on stderr.

Is not about to Terraform Client Debugging, panic errors, go errors, etc. Is useful to provide the logs to Hashicorp. You can run the following command: `export TF_LOG=TRACE`.

You can set `TF_LOG` to one of the log levels `TRACE`, `DEBUG`, `INFO`, `WARN` or `ERROR` to change the verbosity of the logs. `TRACE` is the most verbose and it is the default if `TF_LOG` is set to something other than a log level name.

> **Note** 
> When `TF_LOG_PATH` is set, `TF_LOG` must be set in order for any logging to be enabled, and `TF_LOG_PATH` point to a specific file (not directory), for example `TF_LOG_PATH=./terraform.log`

## Miscellaneous
-   A Terraform Enterprise install that is provisioned on a network that does not have Internet access is generally known as an _air-gapped_ install. These types of installs require you to pull updates, providers, etc. from external sources vs. being able to download them directly.
-   Terraform Enterprise requires _PostgresSQL_ for a clustered deployment.
-   Some backends thtr are supported: (1) Terraform Enterprise, (2) Consul, (3) AWS S3, (4) Artifactory.
-   Terraform Cloud supports the following VCS providers: (1) GitHub, (2) Gitlab, (2) Bitbucket and (4) Azure DevOps
-   The existence of a provider plugin found locally in the working directory does not itself create a provider dependency. The plugin can exist without any reference to it in Terraform configuration.
-   Function `index` finds the element index for a given value in a list starting with index 0.
-   HashiCorp style conventions suggest you that align the equals sign for consecutive arguments for easing readability for configurations
    
    ```
    ami           = "abc123"
    instance_type = "t2.micro"
    ```
    
-   Terraform can limit the number of concurrent operations as Terraform [walks the graph](https://www.terraform.io/docs/internals/graph.html#walking-the-graph) using the [`-parallelism=n`](https://www.terraform.io/docs/commands/plan.html#parallelism-n) argument. The default value for this setting is `10`. This setting might be helpful if you're running into API rate limits.
-   HashiCorp style conventions state that you should use two spaces between each nesting level to improve the readability of Terraform configurations.
-   Terraform supports the `#`, `//`, and `/*..*/` for commenting Terraform configuration files. Please use them when writing Terraform so both you and others who are using your code have a full understanding of what the code is intended to do.
-   The `terraform console` command provides an interactive console for evaluating [expressions](https://www.terraform.io/docs/configuration/expressions.html) such interpolations. <https://www.terraform.io/docs/commands/console.html>.

[^1]: https://medium.com/better-programming/how-terraform-works-a-visual-intro-6328cddbe067
[^2]: https://medium.com/@mfundo/terraform-modules-illustrate-26cbc48be83a
