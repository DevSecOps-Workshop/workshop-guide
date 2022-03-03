+++
title = "Integrating ACS into the Pipeline"
weight = 30
+++

## Finally: Putting the Sec in DevSecOps!

{{% notice tip %}}
There are basically two ways to interface with ACS. The UI, which focuses on the needs of the security team, and a separate "interface" for developers to integrate into their existing toolset (CI/CD pipeline, consoles, ticketing systems etc): The `roxctl` commandline tool. This way ACS provides a familiar interface to understand and address issues that the security team considers important.
{{% /notice %}}

ACS policies can act during the CI/CD pipeline to identify security risk in images before they are run as a container.

## Integrate Image Scan into the Pipeline
You should have created and build a custom policy in ACS and tested it for triggering violations. Now you will integrate it into the build pipeline.


### Let's go: Prepare `roxctl`

Build-time policies require the use of the `roxctl` command-line tool which is available for download from the ACS Central UI, in the upper right corner of the dashboard. `Roxctl` needs to authenticate to **ACS Central** to do anything. It can use either username and password API tokens to authenticate against Central. It's good practice to use a token so that's what we'll do.

### Create the `roxctl` token

In the **ACS portal**:
- Navigate to **Platform Configure > Integrations**.
- Scroll down to the **Authentication Tokens** category, and select **API Token**.
- Click **Generate Token**. Enter the name `pipeline` for the token and select the role **Admin**.
- Select **Generate**
- Save the contents of the token somewhere!

### Create OCP secret with token
Change to the **OpenShift Web Console** and create a secret with the API token in the project your pipeline lives in:
- In the UI switch to your Project
- Create a new key/value `Secret` named **roxsecrets**
- Introduce these key/values into the secret:
  - **rox_central_endpoint**: \<the URL to your **ACS Portal**>
    - It should be something like central-stackrox.apps.cluster-psslb.psslb.sandbox555.opentlc.com:443
  - **rox_api_token**: \<the API token you generated>

### Create a Scan Task
You are now ready to create a new pipeline task that will use `roxctl` to scan the image build in your pipeline before the deploy step:
- In the OCP UI, make sure you are still in the project with your pipeline and the secret `roxsecrets`
- Go to **Pipelines->Tasks**
- Click **Create-> ClusterTask**
- Replace the YAML displayed with this:
```yaml
apiVersion: tekton.dev/v1beta1
kind: ClusterTask
metadata:
  name: rox-image-check
spec:
  params:
    - description: >-
        Secret containing the address:port tuple for StackRox Central (example -
        rox.stackrox.io:443)
      name: rox_central_endpoint
      type: string
    - description: Secret containing the StackRox API token with CI permissions
      name: rox_api_token
      type: string
    - description: 'Full name of image to scan (example -- gcr.io/rox/sample:5.0-rc1)'
      name: image
      type: string
    - description: Use image digest result from s2i-java build task
      name: image_digest
      type: string
  results:
    - description: Output of `roxctl image check`
      name: check_output
  steps:
    - env:
        - name: ROX_API_TOKEN
          valueFrom:
            secretKeyRef:
              key: rox_api_token
              name: $(params.rox_api_token)
        - name: ROX_CENTRAL_ENDPOINT
          valueFrom:
            secretKeyRef:
              key: rox_central_endpoint
              name: $(params.rox_central_endpoint)
      image: registry.access.redhat.com/ubi8/ubi-minimal:latest
      name: rox-image-check
      resources: {}
      script: >
        #!/usr/bin/env bash

        set +x

        curl -k -L -H "Authorization: Bearer $ROX_API_TOKEN"
        https://$ROX_CENTRAL_ENDPOINT/api/cli/download/roxctl-linux --output
        ./roxctl  > /dev/null; echo "Getting roxctl"

        chmod +x ./roxctl  > /dev/null

        ./roxctl image check -c Workshop --insecure-skip-tls-verify -e $ROX_CENTRAL_ENDPOINT
        --image $(params.image)@$(params.image_digest)
```
Take your time to **understand** the Tekton task definition:
- First some parameters are defined, it's important to understand some of these are taken or depend on the build task that run before.
- The script action pulls the `roxctl` binary into the pipeline workspace so you'll always have a version compatible with your ACS version.
- The most important bit is the `roxctl` execution, of course.
  - it executes the `image check` command
  - only checks against policies from category **Workshop** that was created above. This way you can check against a subset of policies!
  - defines the image to check and it's digest

### Add the Task to the Pipeline

- Now add the **rox-image-check** task to your pipeline between the **build** and **deploy** steps.

After you added it you have to fill in values for the parameters the task defines. Click the task, a form with the parameters will open, fill it in:
  - **rox_central_endpoint**: `roxsecrets`
  - **rox_api_token**: `roxsecrets`
  - **image**: `image-registry.openshift-image-registry.svc:5000/workshop-int/workshop`
    - Adapt the Project name if you changed it
  - **image_digest**: $(tasks.build.results.IMAGE_DIGEST)
    - This variable takes the result of the **build** task and uses it in the scan task.
  - Click **Save**

{{< figure src="../images/pipeline.png?width=30pc&classes=border,shadow" title="Click image to enlarge" >}}

## Test the Scan Task
With our custom **System Policy** still not set to `enforce` we first are going to test the pipeline integration. Start the pipeline with Java **Version** `java-old-image`
- Expected Result:
  - The `rox-image-check` task should succeed, but if you have a look at the output (click the task in the visual representation) you should see that the **build violated our policy**!

To test the fixed image, just start the task with the default (latest) Java version again.
- Expected Result:
  - The `rox-image-check` task should succeed, if you have a look at the output you should see **no policy violation**!

## Enforce the Policy
The last step is to enforce the System Policy. If the policy is violated the pipeline should be stopped and the application should not be deployed.

- Edit the **System Policy** in **ACS Portal** and set enforcement to **On** for the stages **Build** and **Deploy**
- Run the pipeline again, first with Java **Version** `java-old-image` and then with the latest default version.
- Expecte results:
  - We are sure you know by now what to expect!
  - The pipeline should fail with the old Java image version and succeed with the latest image version!
