---
title: Lab 3 - Install Dynatrace Agent
workshops: dynatrace_openshift
workshop_weight: 13
layout: lab
---

# Deploy the Dynatrace OneAgent Operator on OpenShift 4.4+ 

<blockquote>
Prerequisites:
1) If no existing account, sign-up for a trial with Dynatrace at https://www.dynatrace.com/trial/

2) Dynatrace API Token
   from Settings > Integration > Dynatrace API > Generate Token
   Activate the following settings:
      * Access problem and event feed, metrics, and topology
      * Read log content
      * Write configuration
   
3) Dynatrace PaaS token PaaS (used to download OneAgent and ActiveGate installers)
   from Settings >  Integration > Platform as a Service > Generate Token
   
4) apiUrl -  URL to the API of your Dynatrace environment. 
   In Dynatrace SaaS it will look like https://ENVIRONMENT_ID.live.dynatrace.com/api
      e.g. https://eye15053.live.dynatrace.com/api
   In Dynatrace Managed it will look like https://DOMAIN/e/ENVIRONMENT_ID/api
      e.g. https://mtx879.dynatrace-managed.com/e/65c5d8d9-9cb4-42bb-b24d-4f7817eaf3a6/api
</blockquote>

<blockquote>
The first step is to define the new project where we'll deploy the Dynatrace OneAgent pods
</blockquote>

```bash
oc new-project dynatrace
```

<blockquote>
Using the information you gathered in the pre-requisites referenced above, define the secret that will be used to provision the operator.  You may update this secret at any time to rotate the tokens.
</blockquote>

```bash
oc -n dynatrace create secret generic oneagent \
--from-literal="apiToken=<API token>" --from-literal="paasToken=<PaaS token>"
```

<blockquote>
Optional: Check the latest release at 
https://github.com/Dynatrace/dynatrace-oneagent-operator/branches/active
</blockquote>

<blockquote>
Apply the latest configuration from Dynatrace:
</blockquote>

```bash
oc apply -f \
https://github.com/Dynatrace/dynatrace-oneagent-operator/releases/latest/download/openshift.yaml
```
<blockquote>
You should see an output *similar* to the following:
</blockquote>

```bash
customresourcedefinition.apiextensions.k8s.io/oneagentapms.dynatrace.com created
customresourcedefinition.apiextensions.k8s.io/oneagents.dynatrace.com created
serviceaccount/dynatrace-oneagent created
serviceaccount/dynatrace-oneagent-operator created
serviceaccount/dynatrace-oneagent-webhook created
role.rbac.authorization.k8s.io/dynatrace-oneagent-operator created
role.rbac.authorization.k8s.io/dynatrace-oneagent-webhook created
clusterrole.rbac.authorization.k8s.io/dynatrace-oneagent-operator created
clusterrole.rbac.authorization.k8s.io/dynatrace-oneagent-webhook created
rolebinding.rbac.authorization.k8s.io/dynatrace-oneagent-operator created
rolebinding.rbac.authorization.k8s.io/dynatrace-oneagent-webhook created
clusterrolebinding.rbac.authorization.k8s.io/dynatrace-oneagent-operator created
clusterrolebinding.rbac.authorization.k8s.io/dynatrace-oneagent-webhook created
service/dynatrace-oneagent-webhook created
deployment.apps/dynatrace-oneagent-operator created
deployment.apps/dynatrace-oneagent-webhook created
securitycontextconstraints.security.openshift.io/dynatrace-oneagent-privileged created
mutatingwebhookconfiguration.admissionregistration.k8s.io/dynatrace-oneagent-webhook created
```

<blockquote>
Create the default custom resource for the Dynatrace OneAgent:
</blockquote>

```bash
curl -o cr.yaml https://raw.githubusercontent.com/Dynatrace/dynatrace-oneagent-operator/master/deploy/cr.yaml
```

<blockquote>
Update the custom resource (cr.yaml) with apiUrl and the name of secret we create above ("oneagent") as shown below

e.g.
apiUrl: https://eye15053.live.dynatrace.com/api
tokens: "oneagent"
</blockquote>

```bash
vi cr.yaml
```

<blockquote>

</blockquote>

```
apiVersion: dynatrace.com/v1alpha1
kind: OneAgent
metadata:
  # a descriptive name for this object.
  # all created child objects will be based on it.
  name: oneagent
  namespace: dynatrace
spec:
  # dynatrace api url including `/api` path at the end
  # either set ENVIRONMENTID to the proper tenant id or change the apiUrl as a whole, e.q. for Managed
  apiUrl: https://eye15053.live.dynatrace.com/api
  # disable certificate validation checks for installer download and API communication
  skipCertCheck: false
  # name of secret holding `apiToken` and `paasToken`
  # if unset, name of custom resource is used
  tokens: "oneagent"
```

<blockquote>
If you want Dynatrace to monitor OpenShift Service Mesh deployments, set
</blockquote>

```
enableIstio: true
```

<blockquote>
Apply the modified custom resource (cr.yaml)
</blockquote>

```bash
oc apply -f cr.yaml
oneagent.dynatrace.com/oneagent configured
```

<blockquote>
Verify that the operator, webhook, and OneAgent pods have been created and are deploying
</blockquote>

```bash
oc get pods
```

<blockquote>
The outpout should look *similar* to the following:
</blockquote>

```bash
NAME                                           READY   STATUS    RESTARTS   AGE
dynatrace-oneagent-operator-788fd7f5b4-6lt67   1/1     Running   0          4m21s
dynatrace-oneagent-webhook-84747567df-lmltw    2/2     Running   0          4m21s
oneagent-4j9xf                                 0/1     Running   0          102s
oneagent-55p2k                                 0/1     Running   0          106s
oneagent-b7qlb                                 0/1     Running   0          108s
oneagent-jhk2f                                 0/1     Running   0          107s
```

<blockquote>
Check the deployment status of one of the pods:
</blockquote>

```bash
oc logs oneagent-jhk2f
```

<blockquote>
The output should look *similar* to the following:
</blockquote>

```bash
23:19:49 Started agent deployment as a container, PID 1352627.
23:19:49 Downloading agent to /tmp/Dynatrace-OneAgent-Linux.sh via https://eye15053.live.dynatrace.com/api/v1/deployment/installer/agent/unix/default/latest?Api-Token=***&arch=x86&flavor=default
23:20:18 Download complete
23:20:18 Downloaded version: 1.195.161.20200720-160625
23:20:18 Verifying agent installer signature
23:20:21 Verification successful
23:20:21 Deploying to: /mnt/host_root
23:20:21 Starting installer...
23:20:22 Warning: Parameter APP_LOG_CONTENT_ACCESS is deprecated and will be removed in future release. Please use --set-app-log-content-access instead. For details, see https://www.dynatrace.com/support/help/shortlink/oneagentctl
23:20:23 Checking root privileges...
23:20:23 OK
23:20:23 Installation started, version 1.195.161.20200720-160625, build date: 20.07.2020, PID 1352627.
23:20:25 Detected platform: LINUX arch: X86
23:20:25 Detected bitness: 64
23:20:25 Checking free space in /opt/dynatrace/oneagent
23:20:27 Extracting...
23:20:28 Unpacking. This may take a few minutes...
23:20:52 Unpacking complete.
23:20:52 Moving new binaries into lib folders...
23:20:54 User 'dtuser' added successfully.
23:20:57 Non-privileged mode is enabled.
23:20:57 Applying agent configuration
23:20:58 Storing SELinux policy sources in /opt/dynatrace/oneagent/agent.
23:20:58 Installing SELinux Dynatrace module. This may take a while...
23:21:35 dynatrace_oneagent module was successfully installed
```

## Verify that the Dynatrace OneAgent Operator is deployed in both the OpenShift Management Console and the Dynatrace tenant
First verify the deployment via the OpenShift Management Conole

<blockquote>
In the "Developer" interface, ensure that you are in the project "dynatrace" and choose topology to verify that you have three deployments: dynatrace-oneagent-operator, dynatrace-oneagent-webhook, and oneagent
</blockquote>
<img src="../images/ocp-dynatrace-oneagent-operator-1.png" width="500"><br/>

Then verify the deployment via the Dynatrace Tenant interface

<blockquote>
In the Dynatrace tenant, select "Deployment status" and ensure that you have 6 Dynatrace OneAgents deployed across the 6 hosts in the cluster
</blockquote>
<img src="../images/ocp-dynatrace-oneagent-operator-2.png" width="500"><br/>


{{< importPartial "footer/footer.html" >}}
