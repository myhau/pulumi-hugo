---
title: "Provider functions"
meta_desc: Providers may export functions that can be called in a Pulumi program
menu:
  intro:
    parent: resources
    weight: 7
---

A provider may make **functions** available in its SDK as well as resource types. For example, the AWS provider includes the function [`aws.apigateway.getDomainName`](https://www.pulumi.com/registry/packages/aws/api-docs/apigateway/getdomainname/), among many others.

<div><pulumi-examples>
<div><pulumi-chooser type="language" options="typescript,python,go,csharp,java,yaml"></pulumi-chooser></div>
<div>
<pulumi-choosable type="language" values="csharp">

```csharp
using Pulumi;
using Aws = Pulumi.Aws;

class MyStack : Stack
{
    public MyStack()
    {
        var example = Output.Create(Aws.ApiGateway.GetDomainName.InvokeAsync(new Aws.ApiGateway.GetDomainNameArgs
        {
            DomainName = "api.example.com",
        }));
    }
}
```

</pulumi-choosable>
</div>
<div>
<pulumi-choosable type="language" values="go">

```go
package main

import (
	"github.com/pulumi/pulumi-aws/sdk/v5/go/aws/apigateway"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {
		_, err := apigateway.LookupDomainName(ctx, &apigateway.LookupDomainNameArgs{
			DomainName: "api.example.com",
		}, nil)
		if err != nil {
			return err
		}
		return nil
	})
}
```

</pulumi-choosable>
</div>
<div>
<pulumi-choosable type="language" values="java">

```java
package generated_program;

import java.util.*;
import java.io.*;
import java.nio.*;
import com.pulumi.*;

public class App {
    public static void main(String[] args) {
        Pulumi.run(App::stack);
    }

    public static void stack(Context ctx) {
        final var example = Output.of(ApigatewayFunctions.getDomainName(GetDomainNameArgs.builder()
            .domainName("api.example.com")
            .build()));
    }
}
```

</pulumi-choosable>
</div>
<div>
<pulumi-choosable type="language" values="python">

```python
import pulumi
import pulumi_aws as aws

example = aws.apigateway.get_domain_name(domain_name="api.example.com")
```

</pulumi-choosable>
</div>
<div>
<pulumi-choosable type="language" values="typescript">

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

const example = pulumi.output(aws.apigateway.getDomainName({
    domainName: "api.example.com",
}));
```

</pulumi-choosable>
</div>
<div>
<pulumi-choosable type="language" values="yaml">

```yaml
variables:
  example:
    Fn::Invoke:
      Function: aws:apigateway:getDomainName
      Arguments:
        domainName: api.example.com
```

</pulumi-choosable>
</div>
</pulumi-examples></div>

Provider functions are exposed in each language as a regular functions, in two variations:

 1. a function that accepts plain arguments (strings and so on) and returns a Promise, or blocks until the result is available; and,
 2. a function that accepts `Input` values and returns an [Output]({{< relref "/docs/intro/concepts/inputs-outputs" >}}).

The documentation for a provider function will tell you the name and signature for each of the variations.

### Invoke options

Each function also accepts "invoke options", either as an object or as varargs depending on the host
language. The options are as follows:

| Option | Explanation                                                  |
|--------|--------------------------------------------------------------|
| parent | Supply a parent resource, which will be used to determine default providers, and to calculate the protect property. |
| provider | Supply the provider to use explicitly. |
| version | Use exactly this version of the provider plugin. |
| pluginDownloadURL | Download the provider plugin from this URL. The download URL is otherwise inferred from the provider package. |
| _async_ | _This option is deprecated and will be removed in a future release_ |

The `parent` option has a similar purpose to the [parent option]({{< relref "/docs/intro/concepts/resources/options/parent" >}}) used when creating a resource. The parent is consulted when determining the provider to use, and the [`protect`]({{< relref "/docs/intro/concepts/resources/options/protect" >}}) status of any resources created as a result of the function invocation.

The `provider` option gives an explicit provider to use when running the invoked function. This is useful, for example, if you want to invoke a function in each of a set of AWS regions.

The `version` option specifies an exact version for the provider plugin. This can be used when you need to pin to a specific version to avoid a backward-incompatible change.

The `pluginDownloadURL` option gives a URL for fetching the provider plugin. It may be necessary to supply this for third-party packages (those not hosted at [https://get.pulumi.com](https://get.pulumi.com)).

## Getter functions

A special case of functions is the static `get` function, which is available on all resource types, to look up an existing resource’s ID. The `get` function is different from the `import` function. The difference is that, although the resulting resource object’s state will match the live state from an existing environment, the resource will not be managed by Pulumi. A resource read with the `get` function will never be updated or deleted by Pulumi during an update.

You can use the `get` function to consume properties from a resource that was provisioned elsewhere. For example, this program reads an existing EC2 Security Group whose ID is `sg-0dfd33cdac25b1ec9` and uses the result as input to create an EC2 Instance that Pulumi will manage:

{{< chooser language "javascript,typescript,python,go,csharp,java,yaml" >}}

{{% choosable language javascript %}}

```javascript
let aws = require("@pulumi/aws");

let group = aws.ec2.SecurityGroup.get("group", "sg-0dfd33cdac25b1ec9");

let server = new aws.ec2.Instance("web-server", {
    ami: "ami-6869aa05",
    instanceType: "t2.micro",
    securityGroups: [ group.name ], // reference the security group resource above
});
```

{{% /choosable %}}
{{% choosable language typescript %}}

```typescript
import * as aws from "@pulumi/aws";

let group = aws.ec2.SecurityGroup.get("group", "sg-0dfd33cdac25b1ec9");

let server = new aws.ec2.Instance("web-server", {
    ami: "ami-6869aa05",
    instanceType: "t2.micro",
    securityGroups: [ group.name ], // reference the security group resource above
});
```

{{% /choosable %}}
{{% choosable language python %}}

```python
import pulumi_aws as aws

group = aws.ec2.SecurityGroup.get('group', 'sg-0dfd33cdac25b1ec9')

server = aws.ec2.Instance('web-server',
    ami='ami-6869aa05',
    instance_type='t2.micro',
    security_groups=[group.name]) # reference the security group resource above
```

{{% /choosable %}}
{{% choosable language go %}}

```go
import (
    "github.com/pulumi/pulumi-aws/sdk/v4/go/aws/ec2"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
        group, err := ec2.GetSecurityGroup(ctx, "group", pulumi.ID("sg-0dfd33cdac25b1ec9"), nil)
        if err != nil {
            return err
        }
        server, err := ec2.NewInstance(ctx, "web-server", &ec2.InstanceArgs{
            Ami:            pulumi.String("ami-6869aa05"),
            InstanceType:   pulumi.String("t2.micro"),
            SecurityGroups: pulumi.StringArray{group.Name},
        })
        if err != nil {
            return err
        }
        return nil
    })
}
```

{{% /choosable %}}
{{% choosable language csharp %}}

```csharp
using Pulumi;
using Pulumi.Aws.Ec2;
using Pulumi.Aws.Ec2.Inputs;

class MyStack : Stack
{
    public MyStack()
    {
        var group = SecurityGroup.Get("group", "sg-0dfd33cdac25b1ec9");

        var server = new Instance("web-server", new InstanceArgs {
            Ami = "ami-6869aa05",
            InstanceType = "t2.micro",
            SecurityGroups = { group.Name }
        });
    }
}
```

{{% /choosable %}}
{{% choosable language java %}}

```java
public static void stack(Context ctx) {
    var group = SecurityGroup.get("group", Output.of("sg-0dfd33cdac25b1ec9"), null, null);

    var server = new Instance("web-server", InstanceArgs.builder()
        .ami("ami-6869aa05")
        .instanceType("t2.micro")
        .securityGroups(
            group.name().applyValue(v -> List.of(v)))
        .build());
}
```

{{% /choosable %}}
{{% choosable language yaml %}}

```yaml
# YAML does not yet support the `get` function. See
# https://github.com/pulumi/pulumi-yaml/issues/155 for details.
```

{{% /choosable %}}

{{< /chooser >}}

Two values are passed to the `get` function - the logical name Pulumi will use to refer to the resource, and the physical ID that the resource has in the target cloud.

Importantly, Pulumi will never attempt to modify the security group in this example. It simply reads back the state from your currently configured cloud account and then uses it as input for the new EC2 Instance.
