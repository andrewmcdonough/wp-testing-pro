---
ID: 7759
post_title: 'Docker image scanning &#8211; How to implement open source container security (part 2).'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/container-security-docker-image-scanning/
published: true
post_date: 2018-07-16 13:03:56
---
In this article we will cover Docker image scanning with open source container security / image scanning tools. We will explain how to to deploy and setup Docker image scanning: both on private Docker repositories, and as a CI/CD pipeline validation step. We will also explore ways of integrating image scanning with CI/CD tools like Jenkins, Kubernetes runtime configuration features and runtime security tools like Falco.

*This is the second post in a two-part series on Open Source Container Security. The first post focused on <a href="https://sysdigrp2rs.wpengine.com/blog/oss-container-security-runtime/" target="_blank">open source container runtime security</a> using Falco to build a response engine for Kubernetes with security playbooks as FaaS.*

Read on to learn about:

*   [Container Image Scanning][1]
*   [Container Image Scanning Open Source Tools][2]
*   [Open Source Docker Scanning Tool: Anchore Engine][3]
*   [Deploying the Anchore Engine for Docker Image Scanning][4]
*   [Configuring Anchore to Scan your Private Docker Repositories][5]
*   [CI/CD Security: Docker Scanner with Jenkins][6]
*   [Integrating Anchore Engine and Kubernetes for Image Validation][7]
*   [Blocking Forbidden Docker Images or Unscanned Images][8]

## Container Image Scanning {#containerimagescanning}

One of the last steps of the CI (Continuous Integration) pipeline involves building the container images that will be pulled and executed in our environment. Therefore, whether you are building Docker images from your own code or but also when using unmodified third party images, it's important to identify and find any known vulnerabilities that may be present in those images. This process is known as container image scanning.

Docker images are composed of several immutable layers, basically a diff over the previous one adding files and other changes, and each one associated with a unique hash id:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/docker_image_scanner_layers-1024x612.png" alt="Docker Image Scanner Layers" width="619" height="389" class="alignleft size-large wp-image-7764" />][9]

Any new Docker image that you create will probably be based in an existing image (`FROM` statement in the `Dockerfile`). That's why you can leverage this layered design to avoid having to re-scan the entire image every time you make a new one, a change. If a parent image is vulnerable, any other images built on top of that one will be vulnerable too.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/docker_image_scanner_layers2.png" alt="Docker Image Scanner Layers 2" width="800" height="300" class="alignleft size-full wp-image-7765" />][10]

The Docker build process follows a manifest (`Dockerfile`) that includes relevant security information that you can scan and evaluate including the base images, exposed ports, environment variables, entrypoint script, external installed binaries, etc. Btw, don't miss our <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/" target="_blank">Docker security best practices</a> article for more hints in building your `Dockerfiles`.

In a secure pipeline, Docker image scanning should be a mandatory step of your CI/CD process and any image should be scanned and approved before ever entering "Running" state in the production clusters.

The container image scanning process typically includes:

*   Checking the software packages, binaries, libraries, operative system files, etc. against one or more well known vulnerabilities databases. Some Docker scanning tools have a repository containing the scanning results for common Docker images that can be used as a cache to speed up the process.
*   Analyzing the `Dockerfile` and image metadata to detect security sensitive configurations like running as privileged (root) user, exposing insecure ports, using based images tagged with "latest" rather than specific versions for full traceability, etc.
*   User defined policies, or any set of requirements that you want to check for every image, like software packages blacklists, base images whitelists, whether a SUID file has been added, etc.

You can can classify and group the different security issues you might find in an image, assigning different priorities: a warning notification is sufficient for some issues, while others will be severe enough to justify aborting the build.

[tweet_box design="default" float="none"]Implementing #Docker image scanning with @anchore #opensource and @sysdig Falco, holding hands to deploy a complete #container #Kubernetes #CloudNative #security stack[/tweet_box] 

## Container Image Scanning Open-source Tools {#containerimagescanningopensourcetools}

There are several Docker image scanning tools available, and some of the most popular include:

*   **<a href="https://github.com/anchore/anchore-engine" target="_blank">Anchore Engine</a>:** Anchore Engine is an open source image scanning tool. Provides a centralized service for inspection, analysis and applies user-defined acceptance policies to allow automated validation and certification of container images.
*   **<a href="https://github.com/coreos/clair" target="_blank">CoreOS/Clair</a>:** An open source project for the static analysis of vulnerabilities in application containers (currently including appc/Rkt and Docker).
*   **<a href="https://vuls.io/" target="_blank">Vuls.io</a>:** Agent-less Linux vulnerability scanner based on information from NVD, OVAL, etc. It has some container image support, although is not a container specific tool.
*   **<a href="https://www.open-scap.org/" target="_blank">OpenScap</a>:** Suite of automated audit tools to examine the configuration and known vulnerabilities in your software, following the NIST-certified Security Content Automation Protocol (SCAP). Not container specific again, but does include some level of support.

## Open Source Docker Scanning Tool: Anchore Engine {#opensourcedockerscanningtoolanchoreengine}

The flexible user-defined <a href="https://anchore.freshdesk.com/support/solutions/articles/36000020652-policies" target="_blank">policies</a>, breadth of analysis, API and performance characteristics of <a href="https://anchore.com/opensource" target="_blank">Anchore Engine</a> made it our top choice among open source tools available today.

*   Anchore Engine allows developers to perform detailed **analysis** on their container images, run queries, produce **reports** and define **policies** that can be used in CI/CD pipelines.
*   Using Anchore Engine, container images can be downloaded from **Docker V2 compatible container registries**, analyzed and evaluated against user defined policies.
*   It can be accessed directly through a RESTful **API** or via the Anchore **CLI** tool.
*   The scanning includes not just <a href="https://es.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures" target="_blank">CVE</a>-based security scans but also policy-based scans that can include **checks around security, compliance and operational best practices**.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_engine_sources_endpoints.png" alt="Anchore Engine Sources Endpoints" width="1301" height="412" class="alignleft size-full wp-image-7766" />][11]

<p align="center">
  Anchore Engine sources and endpoints
</p>

Anchore Engine architecture is comprised of six components that can be deployed in a single container or scaled out:

*   **API Service:** Central communication interface that can be accessed by code using a REST API or directly using the command line.
*   **Image Analyzer Service:** Executed by the "worker", these Anchore nodes that perform the actual Docker image scanning.
*   **Catalog Service:** Internal database and system state service.
*   **Queuing Service:** Organizes, persists and schedules the engine tasks.
*   **Policy Engine Service:** Policy evaluation and vulnerabilities matching rules.
*   **Kubernetes Webhook Service:** Kubernetes-specific webhook service to validate images before they are spawned.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_engine_architecture.png" alt="Anchore Engine Architecture" width="852" height="449" class="alignleft size-full wp-image-7767" />][12]

<p align="center">
  Anchore Engine architecture
</p>

## Deploying the Anchore Engine for Docker Image Scanning {#deployingtheanchoreenginefordockerimagescanning}

### How to Install Anchore Using docker-compose {#howtoinstallanchoreusingdockercompose}

Maybe the fastest way to get a taste of Anchore Engine is to quickly deploy it locally using a <a href="https://docs.docker.com/compose/" target="_blank">docker-compose</a> stack:

    mkdir anchore
    cd anchore
    curl https://raw.githubusercontent.com/anchore/anchore-engine/master/scripts/docker-compose/docker-compose.yaml > docker-compose.yaml
    mkdir config
    curl https://raw.githubusercontent.com/anchore/anchore-engine/master/scripts/docker-compose/config.yaml > config/config.yaml
    mkdir db
    docker-compose up -d

There are several parameters that you can tune in this `config.yaml` file: log level, listening port, credentials, webhook notifications, etc.

The CLI client is also available as a Docker container that you can pull directly from DockerHub:

    docker pull anchore/engine-cli:latest

Then, we can check that all the Anchore Engine services are up and running and we are ready to go:

    docker run anchore/engine-cli:latest anchore-cli --u admin --p foobar --url http://172.18.0.1:8228/v1 system status
    
    Service simplequeue (dockerhostid-anchore-engine, http://anchore-engine:8083): up
    Service analyzer (dockerhostid-anchore-engine, http://anchore-engine:8084): up
    Service policy_engine (dockerhostid-anchore-engine, http://anchore-engine:8087): up
    Service catalog (dockerhostid-anchore-engine, http://anchore-engine:8082): up
    Service apiext (dockerhostid-anchore-engine, http://anchore-engine:8228): up
    Service kubernetes_webhook (dockerhostid-anchore-engine, http://anchore-engine:8338): up
    
    Engine DB Version: 0.0.7
    Engine Code Version: 0.2.3

### How to Install Anchore Engine in Kubernetes using Helm {#howtoinstallanchoreengineinkubernetesusinghelm}

You can also install Anchore Engine in Kubernetes with <a href="https://github.com/kubernetes/helm" target="_blank">Helm</a> using the <a href="https://github.com/kubernetes/charts/tree/master/stable/anchore-engine" target="_blank">Anchore Engine Helm chart</a>. To do so, execute:

    helm install --name anchore-stack stable/anchore-engine

Once deployed, it will print a list of instructions and useful commands to connect to the service.

Before anything else, we can check that all the pods are up and running:

    kubectl get pods
    NAME                                                  READY     STATUS    RESTARTS   AGE
    anchore-stack-anchore-engine-core-5bf44cb6cd-zxx2k    1/1       Running   0          38m
    anchore-stack-anchore-engine-worker-5f865c7bf-r72vs   1/1       Running   0          38m
    anchore-stack-postgresql-76c87599dc-bbnxn             1/1       Running   0          38m

Then, follow the instructions printed to screen to spawn an ephemeral container that has the `anchore-cli` tool:

    ANCHORE_CLI_USER=admin
    ANCHORE_CLI_PASS=$(kubectl get secret --namespace default anchore-stack-anchore-engine -o jsonpath="{.data.adminPassword}" | base64 --decode; echo)
    kubectl run -i --tty anchore-cli --restart=Always --image anchore/engine-cli --env ANCHORE_CLI_USER=admin --env ANCHORE_CLI_PASS=${ANCHORE_CLI_PASS} --env ANCHORE_CLI_URL=http://anchore-stack-anchore-engine.default.svc.cluster.local:8228/v1/
    / anchore-cli system status

There are many parameters that you can configure directly from the `helm install` command. It is always important to replace the default passwords and configure persistent storage volumes. You can review <a href="https://github.com/kubernetes/charts/blob/master/stable/anchore-engine/values.yaml" target="_blank">values.yaml file</a> for a complete list.

## Configure Anchore to Scan your Private Docker Repositories {#configureanchoretoscanyourprivatedockerrepositories}

Adding private Docker V2 compatible image registries to the Anchore Engine is a pretty straightforward process, regardless of whether they are hosted by you or any of the common cloud registries.

To configure Anchore to scan your private Docker repositories, use the registry subcommand for all the registry-related operations. Its operation is mostly self-describing:

    anchore-cli registry add index.docker.io <user> <password>
    anchore-cli registry list
    Registry               Type             User
    index.docker.io        docker_v2        mateobur

Also replace `index.docker.io` with the URL for your local registry.

Once that's complete, you are ready to start pulling and scanning images from the private register:

    anchore-cli image add index.docker.io/mateobur/secureclient:latest
    Image Digest: sha256:dda434d0e19db72c3277944f92e203fe14f407937ed9f3f9534ec0579ce9cdac
    Analysis Status: analyzed
    Image Type: docker
    Image ID: 5e1be4e7763143e8ff153887d2ae510fe1fee59c9a55392d65f4da73c9626d76
    Dockerfile Mode: Guessed
    Distro: ubuntu
    Distro Version: 18.04
    Size: 127739629
    Architecture: amd64
    Layer Count: 6
    
    Full Tag: index.docker.io/mateobur/secureclient:latest

To pull from a private repository, you will need to add the registry url in the container name. Entering only `<username>/<imagename>` won't work.

Note that some vendor-specific Docker registries may need additional parameters on top of just user/password, see <a href="https://anchore.com/blog/scanning-images-on-amazon-elastic-container-registry/" target="_blank">scanning Amazon Elastic Container Registry (ECR) with Anchore</a>, for example.

## CI/CD Security: Docker Scanner with Jenkins {#cicdsecuritydockerscannerwithjenkins}

Jenkins is an open source automation server with a plugin ecosystem that supports the typical tools that are part of your delivery pipelines. Jenkins helps to automate the CI/CD process. Anchore has been designed to plug seamlessly into a CI/CD pipeline: a developer commits code into the source code management system, like Git. This change triggers Jenkins to start a build which creates a container image, etc.

In a typical workflow, this container image is then run through some automated testing. If an image does not meet the organization's requirements for security or compliance then it doesn't make sense to invest the time required to perform automated tests on the image. A better approach is to "learn fast" by failing the build and returning the appropriate reports back to the developer to address the issues..

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/docker_scanner_with_jenkins.png" alt="Docker scanner with Jenkins" width="2500" height="724" class="alignleft size-full wp-image-7768" />][13]

You can use the "<a href="https://wiki.jenkins.io/display/JENKINS/Anchore+Container+Image+Scanner+Plugin" target="_blank">Anchore Container Image Scanner Plugin</a>" available in the official plugin list that you can access via the Jenkins interface.

Once you have installed the plugin and configured the connection with the Engine (follow the instructions in the link above), you can include the Anchore evaluation as another (mandatory) step of your pipeline.

This way, your catalog will be automatically up to date and you will be able to detect and retract insecure images before they reach your Docker registry and automated functional tests are run.

From the `Manage Jenkins -> Configure System` menu, you need to configure the connection with the Engine API endpoint and credentials:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_docker_scanner_plugin.png" alt="Anchore Docker Scanner Plugin" width="1262" height="323" class="alignleft size-full wp-image-7769" />][14]

And as the last step of your build pipeline, you can write the image name, tags and (optionally) `Dockerfile` path to a workspace local file "anchore_images":

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_docker_scanner_plugin2.png" alt="Anchore Docker Scanner Plugin 2" width="694" height="195" class="alignleft size-full wp-image-7770" />][15]

After that, you can invoke the Anchore container image scanner in the next build step:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_docker_scanner_plugin3.png" alt="Anchore Docker Scanner Plugin 3" width="1399" height="413" class="alignleft size-full wp-image-7771" />][16]

The build will fail if Anchore detects any stop build vulnerabilities. Of course, <a href="https://anchore.com/blog/creating-policies/" target="_blank">you also can define</a> what triggers a stop.

The following is an example of a build process stopped by Anchore Engine, with every vulnerability explained, links to the full description / mitigation procedures, etc:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_docker_scanner_plugin_jenkins_report.png" alt="Anchore Docker Scanner Jenkins Plugin report" width="2500" height="1188" class="alignleft size-full wp-image-7772" />][17]

The results of evaluations such as this are added to you Anchore Engine catalog. You can then use this catalog of approved / rejected Docker images to filter which pods will be accepted by the Kubernetes API, as you will see in the next section, or as an input to Sysdig Falco runtime rules.

## Integrating Anchore Engine and Kubernetes for Image Validation {#integratinganchoreengineandkubernetesforimagevalidation}

The Kubernetes <a href="https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook" target="_blank">ImagePolicyWebhook</a> admission controller allows a backend webhook to make admission decisions, such as whether or not a Pod should be allowed in your cluster.

Anchore Engine provides a webhook service specifically designed to enable this feature.

You can read the complete story <a href="https://testingclouds.wordpress.com/2018/02/26/policy-based-image-validation-for-kubernetes-with-anchore-engine/" target="_blank">here</a>, but for the impatient there is also a <a href="https://github.com/viglesiasce/kubernetes-anchore-image-validator#quick-start" target="_blank">quick start script</a>.

Assuming that `kubectl` and `helm` are available and configured already, run:

    git clone https://github.com/viglesiasce/kubernetes-anchore-image-validator.git
    cd kubernetes-anchore-image-validator/
    ./hack/install.sh

After running the `install.sh` script, you will need to enable the webhook Kubernetes configuration. How-to instructions will be printed on the Helm output:

<script src="https://gist.github.com/680e1cc6efea861bcdf109d823a933b8.js"></script> <noscript>
  <pre><code>
File: Docker_scan_ValidatingWebhookConfiguration.yaml
-----------------------------------------------------

Anchore engine policy validator is now installed.

Create a validating webhook resources to start enforcement:

KUBE_CA=$(kubectl config view --minify=true --flatten -o json | jq &#39;.clusters[0].cluster."certificate-authority-data"&#39; -r)
cat &gt; validating-webook.yaml &lt;&lt;EOF
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingWebhookConfiguration
metadata:
  name: analysis-anchore-policy-validator.admission.anchore.io
webhooks:
- name: analysis-anchore-policy-validator.admission.anchore.io
  clientConfig:
    service:
      namespace: default
      name: kubernetes
      path: /apis/admission.anchore.io/v1beta1/imagechecks
    caBundle: $KUBE_CA
  rules:
  - operations:
    - CREATE
    apiGroups:
    - ""
    apiVersions:
    - "*"
    resources:
    - pods
  failurePolicy: Fail
EOF

kubectl apply -f validating-webook.yaml
</code></pre>
</noscript>

A few minutes after enablement, all the pods will be running on the `anchore` namespace:

    kubectl get pods -n anchore
    NAME                                                 READY     STATUS    RESTARTS   AGE
    analysis-anchore-engine-core-97dc7ccdb-w9bhb         1/1       Running   0          3h
    analysis-anchore-engine-worker-7b5c95b57c-d9dmg      1/1       Running   0          3h
    analysis-anchore-policy-validator-54d598ddb7-6p6ts   1/1       Running   0          3h
    analysis-postgresql-c7df6d66f-xhnrz                  1/1       Running   0          3h

After following the instructions to enable the integration you will be able to see the new webhook configured:

    kubectl get validatingwebhookconfiguration
    NAME                                                     AGE
    analysis-anchore-policy-validator.admission.anchore.io   20h

Anchore will now analyse every image on any Pod scheduled in Kubernetes and containers evaluated as non-secure will never get into the running phase, like this:

    Events:
      Type     Reason        Age                 From                  Message
      ----     ------        ----                ----                  -------
      Warning  FailedCreate  19s (x13 over 41s)  daemonset-controller  Error creating: admission webhook "analysis-anchore-policy-validator.admission.anchore.io" denied the request: Image failed policy check: wordpress:latest

## Block Forbidden Docker Images or Images not Scanned {#blockforbiddendockerimagesorimagesnotscanned}

On one side, before deployment, we have image scanning. On the other side, after deployment, we have runtime security. Is there any possible link between the two? Actually yes, events in each side can be useful on both sides.

For example, let's say you decide to integrate Anchore Engine Docker scanning with Falco runtime security. What if you create **a Sysdig Falco rule that will trigger an alert whenever any of the images that are marked as non-secure by Anchore Engine are spawned** in your nodes?. This can be interesting in many cases, for example if you are not running the Kubernetes webhook, running without Kubernetes or the webhook fails, or if the vulnerability is found once the container is running.

If you already know the user, password and URL to contact the Anchore Engine API, you can use the <a href="https://github.com/draios/falco/tree/dev/integrations/anchore-falco" target="_blank">Sysdig Falco / Anchore integration</a> directly:

    docker run --rm -e ANCHORE_CLI_USER=$ANCHORE_USER \
                      -e ANCHORE_CLI_PASS=$ANCHORE_CLI_PASS \
                      -e ANCHORE_CLI_URL=http://$HOST:$PORT/v1 \
                      sysdig/anchore-falco

This will print a rule that you can directly append to your Falco configuration:

    - macro: anchore_stop_policy_evaluation_containers
      condition: container.image.id in ("52057de6c8d0d0143dfc71fde55e58edaf3ccc5c2212221a614f45283c5ab063", "65bf726222e13b0ceff0bb20bb6f0e99cbf403a7a1f611fdd2aadd0c8919bbcf", "a8a59477268d92f434d86a73b5ea6de9bf7b05d536359413e79da1feb31f87aa", "a5b7afcfdcc878fae7696ea80eecc31114b9c712a1e896cba87a999d255e4a7b", "8753edeb1aa342081179474cfe5449f1b7d101d92d2799a7840da744b9bbb3ca", "857bc7ff918f67a8e74abd01dc02308040f7f7f9b99956a815c5a9cb6393f11f", "8626492fecd368469e92258dfcafe055f636cb9cbc321a5865a98a0a6c99b8dd", "e86d9bb526efa0b0401189d8df6e3856d0320a3d20045c87b4e49c8a8bdb22c1")
    
    - rule: Run Anchore Containers with Stop Policy Evaluation
      desc: Detect containers which does not receive a positive Policy Evaluation from Anchore Engine.
    
      condition: evt.type=execve and proc.vpid=1 and container and anchore_stop_policy_evaluation_containers
      output: A stop policy evaluation container from anchore has started (%container.info image=%container.image)
      priority: INFO
      tags: [container]

You will need to periodically run the integration to re-evaluate the status and update this Falco rule with the successive Docker images you want to alert if they are run in your cluster.

## Conclusions {#conclusions}

Here are some key takeaways on image scanning and runtime security:

*   While runtime security takes place after the deployment, image scanning happens in your CI/CD pipeline, either before publishing the images or once they are in your registry.
*   Although runtime security and image scanning happen at different points in the container lifecycle, there are very interesting links to explore between them, including information about what's inside the containers and their function.
*   There are various open-source tools available that provide the functionality you need to analyze, inspect, and support image scanning. Anchore Engine's flexible user-defined policies, API and performance makes it our recommended choice.

Anchore and all the open source projects we mentioned in the first part: Falco, NATS, Kubeless, Helm and obviously Kubernetes - with Python and Golang code, build together what we consider a great <a href="https://sysdigrp2rs.wpengine.com/blog/oss-container-security-runtime/" target="_blank">open source container security reference stack</a>.

If you want to contribute to this project, there are many ways to do so, including:

*   Creating additional documentation on <a href="http://anchore.com/opensource" target="_blank">the open source Anchore Engine</a>.
*   Extending the <a href="https://sysdigrp2rs.wpengine.com/blog/docker-runtime-security/" target="_blank">library of default Falco rules</a>.
*   Contributing more <a href="https://github.com/draios/falco/tree/dev/integrations/kubernetes-response-engine" target="_blank">security playbooks such as Kubeless FaaS or any other NATS observers</a>.
*   Integrating more tools (PRs are always great!)

Join the discussion on <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig's open source Slack community</a>, #open-source-sysdig or reach out us via Twitter on <a href="https://twitter.com/sysdig" target="_blank">@sysdig</a> and <a href="https://twitter.com/anchore" target="_blank">@anchore</a>!

 [1]: #containerimagescanning
 [2]: #containerimagescanningopensourcetools
 [3]: #opensourcedockerscanningtoolanchoreengine
 [4]: #deployingtheanchoreenginefordockerimagescanning
 [5]: #configureanchoretoscanyourprivatedockerrepositories
 [6]: #cicdsecuritydockerscannerwithjenkins
 [7]: #integratinganchoreengineandkubernetesforimagevalidation
 [8]: #blockforbiddendockerimagesorimagesnotscanned
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/docker_image_scanner_layers.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/docker_image_scanner_layers2.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_engine_sources_endpoints.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_engine_architecture.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/docker_scanner_with_jenkins.png
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_docker_scanner_plugin.png
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_docker_scanner_plugin2.png
 [16]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_docker_scanner_plugin3.png
 [17]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/anchore_docker_scanner_plugin_jenkins_report.png