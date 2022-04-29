# Exercise 5: Configure and Test Cartographer 

> Important Concepts to cover in an overview:
>
> - Overview of Cartographer (templates, supply chains, workloads)

**Important!** Shell commands on this page should be executed from a terminal open in the root directory of the
git repo you cloned in the pre-requisites!

So far we've seen that we can simply build images with Kpack, and we can simply deploy applications with Knative.
But there's a lot of manual work involved:

- We have to define a Kpack image for the source code
- Once the image is built, we need to retrive the new SHA
- We need to deploy or update the Knative service with the new image

We have not talked about how the images and services should be updated if a developer were to commit a code
change. This seems like a job for automation.

In this exercise, we'll take a look at Cartographer - an open source project build for exactly this purpose.
Cartographer is a supply chain choreographer. It is built to allow platform operators to define pre-built
paths to production that developers could easily use.

Cartographer is open source and you can read all about it here: https://cartographer.sh/

TODO - build out this exercise

[&lt;- Previous](Exercise04-Kpack.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Next -&gt;](Exercise99-Cleanup.md)