# YTT Overview

YTT is a YAML templating tool that forms the basis for many of the configuration functions in Tanzu. Many other tools
in the Tanzu and Carvel portfolio rely on YTT, so it is a fundamental tool in this space.

YTT operates in two main modes: templating and overlays.

1. You can create YAML templates that accept input values from a variety of sources. YTT includes a templating language based
   on Google Starlink that can be used to build logic into templates (loops, conditionals, etc.) YTT templates are part of the
   installation of many Tanzu tools. Further, you will most likely need to write YTT templates if you want to create your own
   Cartographer supply chains or if you use the kapp-controller. The template mode is the mode we will explore in this lesson.
2. YTT can also operate in "overlay" mode where it can modify (or patch) existing YAML. This is useful if you need to modify
   YAML that is not configured as a YTT template. If you want to build a kapp-controller package to install some third party tool,
   you may need to use YTT's overlay capabilities to modify an installation YAML to match your cluster configuration.

In templating mode, YTT will take one or more input YAML files, substitute placeholder variables with actual values
from a variety of sources, and output a final result. YTT is "YAML aware" which means that YTT understands the correct
structure of YAML documents and can build valid YAML composed of fragments. If you are familiar with compilers, you
can think that YTT builds an AST (abstract syntax tree) for YAML - which is far more sophisticated than simple string
substitution. This can be incredibly useful if you need to build a single template that can be reused many times (as we
will do with Cartographer).

Full details about YTT are here: https://carvel.dev/ytt/. That page also includes an online "playground" for YTT that
can be very helpful when learning more about YTT. You can access the playground directly
here: https://carvel.dev/ytt/#playground

## Simple Example
Here is a simple example. Suppose we have a YAML file named `MyName.yaml` like this:

```yaml
#@ load("@ytt:data", "data")
my:
  name:
    is: #@ data.values.my_name
```

We can execute YTT with the following command:

```shell
ytt -f MyName.yaml -v my_name=Fred
```

The output looks like this:

```yaml
my:
  name:
    is: Fred
```

All YTT commands look like YAML comments, so YAML templates are usually valid YAML and will pass validation tools.
In the source file, there are two YTT commands:

1. `#@ load("@ytt:data", "data")` - this instructs YTT to load that `data` module and make all the `data` values
   available with the key `data.values` (`values` is built in and provides access to the individual data elements.)
2. `#@ data.values.my_name` - this instructs YTT to insert the data value `my_name` into the YAML at this position

Data values can come from a variety of sources - most commonly from command line flags as shown above, or from a file
as we will show below.

## Multiple Input Files

If you supply more than one input file to YTT, the output will be a single consolidated YAML file. For example, suppose
there are two input files named `MyName.yaml` and `YourName.yaml` as shown below:

```yaml
#@ load("@ytt:data", "data")
my:
  name:
    is: #@ data.values.my_name
```

```yaml
#@ load("@ytt:data", "data")
your:
  name:
    is: #@ data.values.your_name
```

We can execute YTT with the following command:

```shell
ytt -f MyName.yaml -f YourName.yaml -v my_name=Fred -v your_name=Wilma
```

The output looks like this:

```yaml
my:
  name:
    is: Fred
---
your:
  name:
    is: Wilma
```

As you can see, YTT created a single output YAML and substituted the correct values in each input file.

## Using a Values Input File

Rather than specifying data values with command line flags, you can also use a file (commonly named `values.yaml`) to
supply input values. For example, suppose we have the same two input files named `MyName.yaml` and `YourName.yaml` as shown below:

```yaml
#@ load("@ytt:data", "data")
my:
  name:
    is: #@ data.values.my_name
```

```yaml
#@ load("@ytt:data", "data")
your:
  name:
    is: #@ data.values.your_name
```

But now we have another file named `values.yaml` with content as shown below:

```yaml
my_name: Fred
your_name: Wilma
```

We can execute YTT with the following command:

```shell
ytt -f MyName.yaml -f YourName.yaml --data-values-file values.yaml
```

The output looks like this:

```yaml
my:
  name:
    is: Fred
---
your:
  name:
    is: Wilma
```

As you can see, YTT created the same output as before and substituted the correct values in each input file.

## YTT Schema

A schema file can be used to document the variables that are expected in a template, and provide default values for them.

Here is a simple example. Suppose we have a YAML file named `schema.yaml` like this:

```yaml
#@data/values-schema
---
my_name: Barney

#@schema/nullable
your_name: ""
```

This file indicates two variables - our familiar `my_name` and `your_name`. The `my_name` variable has a default value of "Barney".
The `your_name` variable is in a section denoted "nullable" - which means that if no value is specified for that variable, it's value
will be null.

We can try it out with this command:

```shell
ytt -f MyName.yaml -f YourName.yaml -f schema.yaml
```

The output is as follows:

```yaml
my:
  name:
    is: Barney
---
your:
  name:
    is: null
```

You can see that the default values were used for both `my_name` and `your_name`. The "null" for `your_name` isn't great - we'll deal
with that in the next section.

Of course, we can supply values for these variable in the normal way. So this command produces what we've seen before:

```shell
ytt -f MyName.yaml -f YourName.yaml -f schema.yaml --data-values-file values.yaml
```

## YTT Functions

You can define functions in YTT that do a variety of things. One very useful thing to do with a function is to calculate and return a YAML fragment.

Suppose we have an input file named `YTTFunctions.yaml` like this:

```yaml
#@ load("@ytt:data", "data")

#@ def labels():
type: Name
generated: true
#@ end

#@ def/end name(name):
name:
  #! intentional blank line below ignored by YTT

  is: #@ name
  labels: #@ labels()

my: #@ name(data.values.my_name)
#@ if/end data.values.your_name:
---
your: #@ name(data.values.your_name)
```

There's a lot to unpack here!

1. Functions begin with `def function_name(parameters...):` and end with `end`. On line 3 we define a function named `labels`
   that accepts no parameters. This function returns a YAML fragment with two nodes - two hard coded labels.
2. On line 8 we define a function named `name` that accepts one parameter - also called `name`. This function looks strange
   because of the usage of `def/end`. This is a shortcut we can use when the function returns a single thing. The definition of "single thing"
   can be elusive and confusing. In this case, the function is returning a single YAML node. Note that the end of the node is NOT
   the blank line on line 11. Rather it is the logical end of the node on line 13. This illustrates that YTT is "YAML aware". Also,
   please don't write functions like this in real life!
3. On line 10 we have a YTT comment (starts with `#!`) YTT will strip comments in the resulting YAML.
4. On line 12 we are using the parameter passed into the function
5. On line 13 we are calling the `labels` function. YTT is smart enough to know that the YAML fragment returned from the `labels`
   function should be a child of the node on line 13. We don't need to worry about indenting and formatting - YTT will do it for us
6. On line 15 we call the `name` function passing in an expected input value. Again, the resulting YAML fragment will be properly
   indented and formatted as a child of the node on line 15.
7. On line 16 there is an "if" statement using the same type of shortcut we saw before as `if/end`. If we need more than one thing
   output from the "if" statement we can move the "end" to a separate line as with functions. This "if" statement checks for a null value
   (which is our default for `your_name`) - it will only write the YAML fragment if the value is not null.

We can execute YTT with the following command:

```shell
ytt -f YTTFunctions.yaml -f schema.yaml --data-values-file values.yaml
```

The output will look like this:

```yaml
my:
  name:
    is: Fred
    labels:
      type: Name
      generated: true
---
your:
  name:
    is: Wilma
    labels:
      type: Name
      generated: true
```

Is that what you expected? What happens if we omit the values file?

```shell
ytt -f YTTFunctions.yaml -f schema.yaml
```

Now we just get the first name with the default value:

```yaml
my:
  name:
    is: Barney
    labels:
      type: Name
      generated: true
```

## YTT Built-In Functions

YTT comes with many built-in functions for building more complex YAML. We will show one more example using the `json` and `base64`
modules to create a Kubernetes secret.

Suppose we have an input file named `Secret.yaml` like this:

```yaml
#@ load("@ytt:data", "data")
#@ load("@ytt:json", "json")
#@ load("@ytt:base64", "base64")
apiVersion: v1
kind: Secret
metadata:
  name: #@ data.values.registry.secret.name
type: kubernetes.io/dockerconfigjson
data:
  #@ registry_creds = {"username": data.values.registry.username, "password": data.values.registry.password}
  #@ encoded_creds = base64.encode(json.encode({"auths": {data.values.registry.server: registry_creds}}))
  .dockerconfigjson: #@ encoded_creds
```

This file loads the normal `data` module we are used to, but also loads `json` and `base64` modules. These modules
include encoding functions we can use to encode JSON (in case a supplied password has any special characters), and
then we can Base64 encode a string for setting the secret value. You can also see that we declare local variables
names `registry_creds` and `encoded_creds` using YTT.

Now we can combine this file with a `SecretValues.yaml` file like this:


```yaml
registry:
  server: docker.io
  username: fred
  password: password1234
  secret:
    name: my-secret
```

We can execute YTT with the following command:

```shell
ytt -f Secret.yaml --data-values-file SecretValues.yaml
```

The output looks like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJkb2NrZXIuaW8iOnsicGFzc3dvcmQiOiJwYXNzd29yZDEyMzQiLCJ1c2VybmFtZSI6ImZyZWQifX19
```

You can see that YTT has composed a properly encoded secret for the Kubernetes secret definition.

Normally, YTT does not save its output anywhere - it just pipes the output to stdout.
You can pipe YTT output to `kubectl` and create the secret in a single command:

```shell
ytt -f Secret.yaml --data-values-file SecretValues.yaml | kubectl create -f-
```

Many of the Tanzu configuration commands make use of YTT to perform functions like this. In addition, the Kapp Controller and
Cartographer both support YTT templating in their CRDs as we shall see shortly.

[Next (Kbld Overview) -&gt;](../kbld/README.md)
