+++
title = "The DevSecOps Workshop"
weight = 1
+++

## Intro

This is the storyline you'll follow:

- Create application using the browser based development environment [Red Hat OpenShift Dev Spaces](https://developers.redhat.com/products/openshift-dev-spaces/overview)
- Setting up the **Inner Development Loop** for the individual developer
  - Use the cli tool [odo](https://developers.redhat.com/products/odo/overview) to create, push, change apps on the fly
- Setting up the **Outer Development Loop** for the team CI/CD
  - Learn to work with OpenShift Pipelines based on Tekton
  - Use OpenShift GitOps based on ArgoCD
- Secure your app and OpenShift cluster with **ACS**
  - Introduction to ACS
  - Example use cases
  - Add ACS scanning to Tekton Pipeline

## What to Expect

{{% notice tip %}}
This workshop is for intermediate OpenShift users. A good understanding of how OpenShift works along with hands-on experience is expected. For example we will not tell you how to log in with `oc` to your cluster or tell you what it is... ;)
{{% /notice %}}

We try to balance guided workshop steps and challenging you to use your knowledge to learn new skills. This means you'll get detailed step-by-step instructions for every new chapter/task, later on the guide will become less verbose and we'll weave in some challenges.

## Workshop Environment

To run this workshop you basically need a fresh and empty OpenShift 4.10 cluster with cluster-admin access. In addition you will be asked to use the `oc` commandline client for some tasks.

### As Part of a Red Hat Workshop

As part of the workshop you will be provided with freshly installed OpenShift 4.10 clusters. Depending on attendee numbers we might ask you to gather in teams. Some workshop tasks must be done only once for the cluster (e.g. installing Operators), others like deploying and securing the application can be done by every team member separately in their own Project. This will be mentioned in the guide.

You'll get all access details for your lab cluster from the facilitators. This includes the URL to the OpenShift console and information about how to SSH into your bastion host to run `oc` if asked to.

### On Your Own

As there is not special setup for the OpenShift cluster you should be able to run the workshop with any 4.10 cluster of you own. Just make sure you have cluster admin privileges.

This workshop was tested with these versions :

- Red Hat OpenShift : 4.10.36
- Red Hat Advanced Cluster Security for Kubernetes: 3.74.1
- Red Hat OpenShift Dev Spaces : 3.5.0
- Red Hat OpenShift Pipelines: 1.6.4
- Red Hat OpenShift GitOps: 1.5.1
- Red Hat Quay: 3.8.5
- Red Hat Quay Bridge Operator: 3.7.11
- Red Hat Data Foundation : 4.10.11
- Gitea Operator 1.3.0

## Workshop Flow

We'll tackle the topics at hand step by step with an introduction covering the things worked on before every section.

## And finally a sprinkle of JavaScript magic

You'll notice placeholders for cluster access details, mainly the **part of the domain that is specific to your cluster**. There are two options:

- Whenever you see the placeholder `<DOMAIN>` replace it with the value for your environment
  - This is the part to the right of `apps.` e.g. for `console-openshift-console.apps.cluster-t50z9.t50z9.sandbox4711.opentlc.com` replace <DOMAIN> with `cluster-t50z9.t50z9.sandbox4711.opentlc.com`
- Use a query parameter in the URL of this lab guide to have all occurences replaced automagically, e.g.:
  - http://devsecops-workshop.github.io/?domain=cluster-t50z9.t50z9.sandbox4711.opentlc.com
  - You can use the Link Generator in the next chapter to create the URL for you

## URL Generator for Custom Lab Guide

Enter your Openshift url after the `apps` part (e.g. `cluster-t50z9.t50z9.sandbox4711.opentlc.com` ) and click the button to generate a link that will customize your lab guide.

Click the generated link once to apply it the the current guide.

{{< rawhtml >}}

<script>
  function get_domain() {
  var domain = document.getElementById("domain").value;
  var url = window.location.href + "?domain=" +  domain;
  var a = document.createElement('a');
  var linkText = document.createTextNode(url);
      a.appendChild(linkText);
      a.title = "Custom Lab Guide";
      a.href = url;
      a.id = "dynamicUrl"

  var parentElem = document.getElementById("dynamicLink");
  var elem = parentElem.childNodes.item("dynamicUrl");
  if (elem != null)
  {
    console.log("Replacing in -> " + parentElem + " " + elem);
    
    parentElem.replaceChild(a, elem);
  }
  else
  {
   
    parentElem.appendChild(a);

  }  


  }
</script>
<input type="text" id="domain" name="domain">
<a class="btn btn-default" type = "submit" onclick = "get_domain();" id="skip"  > Generate URL </a>
<p id="dynamicLink"></p>

{{< /rawhtml >}}

## Current Domain replacement

Check to see if replacement is active -> `<DOMAIN>`
