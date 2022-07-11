# YTT Overview

YTT is a YAML templating tool that forms the basis for many of the configuration functions in Tanzu. You
can think of it as a YAML aware string substitution engine. It will take one or more input YAML files,
substitute placeholder variables with actual values from a variety of sources, and output a final result.
This can be incredibly useful if you need to build a single template that can be reused many times (as we will
do with Cartographer).

Full details about YTT are here: https://carvel.dev/ytt/

## Simple Example
Here is a simple example. Suppose we have a YAML file named `MyName.yaml` like this:

```yaml
#@ load("@ytt:data", "data")
my:
  name:
    is: #@ data.values.my.name
```

We can execute YTT with the following command:

```shell
ytt -f MyName.yaml -v my.name=Fred
```

The output looks like this:

```yaml
my:
  name:
    is: Fred
```

All YTT commands look like YAML comments, so YAML templates are always valid YAML and will pass validation tools.
In the source file, there are two YTT commands:

1. `#@ load("@ytt:data", "data")` - this instructs YTT to load that `data` modules and make all the `data` values available with the key `data.values` (`values` is built in and provides access to the individual data elements.)
1. `#@ data.values.my.name` - this instructs YTT to insert the data value `my.name` into the YAML at this position

Data values can come from a variety of sources - most commonly from command line flags as shown above, or from a file as we
will show below.

## Multiple Input Files

If you supply multiple input files to YTT, the output will be a single consolidated YAML file. For example, suppose
there are two input files named `MyName.yaml` and `YourName.yaml` as shown below:

```yaml
#@ load("@ytt:data", "data")
my:
  name:
    is: #@ data.values.my.name
```


```yaml
#@ load("@ytt:data", "data")
your:
  name:
    is: #@ data.values.your.name
```

We can execute YTT with the following command:

```shell
ytt -f MyName.yaml -f YourName.yaml -v my.name=Fred -v your.name=Wilma
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
    is: #@ data.values.my.name
```

```yaml
#@ load("@ytt:data", "data")
your:
  name:
    is: #@ data.values.your.name
```

But now we have another file named `values.yaml` with content as shown below:

```yaml
my:
  name: Fred
your:
 name: Wilma
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

As you can see, ytt created the same output as before and substituted the correct values in each input file.

## YTT Functions

YTT comes with many functions for building more complex yaml. There are `ifs`, `loops`, etc. You can also substitute
entire YAML documents into other documents and YTT will handle all the formatting, indenting, etc.

We will show one more example using the `json` and `base64` modules to create a Kubernetes secret.

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
then we can base64 encode a string for setting the secret value. You can also see that we declare local variables
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

Many of the Tanzu configuration commands make use of YTT to perform functions like this. In addition, Kapp and Cartographer
both support YTT templating in their CRDs as we shall see shortly.
