#  OpenTelemetry Distributed Tracing with Jaeger Observability Platform leveraging Cisco Intersight Service for Terraform and Cisco Intersight Cloud Orchestrator
## Contents
        Use Case

        Pre-requisites

        Pre provisioning Steps

            Step 1: Intersight Target configuration for AppDynamics and on prem entities

            Step 2: Setting up TFCB Workspaces

            Step 3: Share variables with a Global Workspace

            Step 4: Prepping infrastructure & platform for application deployment

        Deploy Jaeger OpenTelemetry components

        Instrumentation of TeaStore cloud native application with OpenTelemetry agents and Deployment

        Generate Application load and view Trace data in Jaeger

        Interfacing with AppDynamics Controller API for De-provisioning
        
            Step 10: Undeploy application services

            Step 11: Undeploy AppDynamics Cluster Agent

            Step 12: Deprovision k8s cluster

### Use Case

* As a DevOps and App developer, use IST (Intersight Service for Terraform) to enable existing containerized k8s services for Jaeger Distributed Tracing 

This use case addresses the third flow in the diagram below: 

![alt text](https://github.com/prathjan/images/blob/main/tomflow3.png?raw=true)

### Pre-requisites

1. You will set up the k8s cluster to host the application containers. You will follow the directions in this Code Exchange entry to create a IKS (Intersight Kubernetes Service) k8s cluster:

https://developer.cisco.com/codeexchange/github/repo/CiscoDevNet/intersight-tfb-iks

2. Sign up for a user account on Intersight.com. You will need Premier license as well as IWO license to complete this use case. 

3. Sign up for a TFCB (Terraform for Cloud Business) at https://app.terraform.io/. Log in and generate the User API Key. You will need this when you create the TF Cloud Target in Intersight.

![alt text](https://github.com/prathjan/images/blob/main/repo.png?raw=true)

### Step 1: Intersight Target configuration for AppDynamics and on prem entities

You will log into your Intersight account and create the following targets. Please refer to Intersight docs for details on how to create these Targets:

    Assist

    vSphere

    AppDynamics

    TFC Cloud

    TFC Cloud Agent - When you claim the TF Cloud Agent, please make sure you have the following added to your Managed Hosts. This is in addition to other local subnets you may have that hosts your kubernetes cluster like the IPPool that you may configure for your k8s addressing:
    NO_PROXY URL's listed:

            github-releases.githubusercontent.com

            github.com

            app.terraform.io

            registry.terraform.io

            releases.hashicorp.com

            archivist.terraform.io

### Step 2: Setting up TFCB Workspaces

1. 

You will set up the following workspaces in TFCB and link to the VCS repos specified. 

    TfIks-JGlobal -> https://github.com/CiscoDevNet/tfiksglobalj

    TfIks-Jaeger -> https://github.com/CiscoDevNet/tfjaeger

    TfIks-JApp -> https://github.com/CiscoDevNet/tfjaegertea

    TfIks-JHost -> https://github.com/CiscoDevNet/tfikshost

    TfIks-JLoad -> https://github.com/CiscoDevNet/tfiksload


2. 

You will set up the TfIks-JGlobal workspace here. 

You will set the following variables:

    appport - eg. 30080	for the Teastore app

    privatekey - ssh privatekey that was used to create  your k8s cluster

    instrapp - heln chart for the OTEL instrumented app

Please also set this workspace to share its data with other workspaces in the organization by enabling Settings->General Settings->Share State Globally.

3. 

You will set up the TfIks-Jaeger workspace here

Set Execution Mode as Agent and select the TF Cloud Agent that you have provisioned.

You will set the following variables:

    ikswsname - eg. sb_iks, which is the workspace that created the IKS k8s cluster

    org - TFCB organization like "CiscoDevNet" or "Lab14"

4. 

You will set up the TfIks-JApp workspace here.

Set Execution Mode as Agent and select the TF Cloud Agent that you have provisioned.

You will set the following variables:

    ikswsname - eg. sb_iks, which is the workspace that created the IKS k8s cluster	

    org - eg. Lab14, which is the TFCB organization	

    globalwsname - TfIks-JGlobal 

5. 

You will set up the TfIks-JHost workspace here.

Set Execution Mode as Agent and select the TF Cloud Agent that you have provisioned.

You will set the following variables:

    org - eg. Lab14	

    ikswsname - eg. sb_iks	


6. 

You will set up the TfIks-JLoad workspace here.

Set Execution Mode as Agent and select the TF Cloud Agent that you have provisioned.

You will set the following variables:

    org - TFCB organization like "CiscoDevNet" or "Lab14"

    ikswsname - eg. sb_iks

    globalwsname - TfIks-JGlobal

    hostwsname	- TfIks-JHost

    trigcount - some random integer to trigger the load

### Step 3: Share variables with a Global Workspace

Execute the TfIks-JGlobal TFCB workspace to setup the global variables for other workspaces. Check for a sucessful Run before progressing to the next step.
        
### Step 4: Prepping for application deployment

Also, Execute the TfIks-JHost TFCB workspace to setup the master node to run some of the utility functions needed in this deployment. This will be needed to generate app load later.

Check for a sucessful Run before progressing to the next step. Also, make a note of the master node IP address.

### Step 5: Deploy Jaeger All-In-One Distributed Tracing Platform

Execute TfIks-Jaeger workdpace to helm install the Jaeger Observability Platform for Distributed Tracing. This will set up the Jaeger OTel Oservability Platform for k8s.

### Step 6: Deploy the OTel instrumented cloud native app

Execute the TfIks-JApp TFCB workspace to deploy the OTel enabled app. This is confiured to send its traces to the Jaeger OTel collector.

View the application deployment status at:

http://<master_node_ip>:<appport>/tools.descartes.teastore.webui/status

![alt text](https://github.com/prathjan/images/blob/main/teastatus.png?raw=true)

View the application at:

http://<master_node_ip>:<appport>/tools.descartes.teastore.webui/

![alt text](https://github.com/prathjan/images/blob/main/tea.png?raw=true)

### Step 7: Generate app load

Execute the TfIks-JLoad TFCB workspace to generate app load

### Step 8: View distributed tracing in Jaeger Obs Platform

Get the jaeger-query LB_IP with: kubectl get svc -l app=jaeger

View the distributed traces for the services at: http://<lb_ip>:16686

![alt text](https://github.com/prathjan/images/blob/main/jaeger.png?raw=true)

            
### Undeploy applications and deprovision infrastructure

Destroy the TFCB workspaces in this order:

TfIks-JLoad

TfIks-JApp

TfIks-Jaeger

TfIks-JHost

TfIks-JGlobal


