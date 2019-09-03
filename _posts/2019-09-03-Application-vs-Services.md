---
title: "What is the difference between Application and Services?"
published: true
---

Often times it is very confusing to distinguish between - "What is an
 Application?" and "What is a Service?" and what should I be saying 
 from a terminology perspective? 
  
### What is a Service? 
* A service is something **that does just one thing** (mostly!)
* Does **not** have a UI
* Exposes an API typically

### What is an App/Application?
* Does **many** things (mostly!)
* Does **have** a **UI** (WebUI, Mobile, CLI, etc)

Application calls multiple services behind the scenes. Say 
<https://www.google.com> or <http://amazon.com> - WebUI would call a bunch of services to serve the page with content.

### Container Perspective
With Cloud-Containerization (Kubernetes) becoming ubiquitous for all 
 developments; it would be safe to say with the above that 
 Kubernetes-cluster  will not host Applications, but will host 
 Services that Applications will consume!
 
### An illustration with a diagram:

<img src="/assets/images/ApplicationVsServices-Illustration.png" 
width="700" height="300" />

In _Summary_, if it is a UI and does many things 
then it is an App/Application and if it is no-UI and does one thing 
only and one thing well then it is a Service.

### Reference:
* [Kubernetes - Scheduling the Future at Cloud Scale](https://assets
.openshift.com/hubfs/pdfs/Kubernetes_OpenShift.pdf?hsLang=en-us)


 
