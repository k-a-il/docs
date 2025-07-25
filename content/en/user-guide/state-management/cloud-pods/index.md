---
title: "Cloud Pods"
weight: 2
description: Get started with Cloud Pods to manage the state of your LocalStack instance state
tags: ["Base"]
---

## Introduction

Cloud pods are persistent state snapshots of your LocalStack instance that can easily be stored, versioned, shared, and restored.
Cloud Pods can be used for various purposes, such as:

- Save and manage snapshots of active LocalStack instances.
- Share state snapshots with your team to debug collectively.
- Automate your testing pipelines by pre-seeding CI environments.
- Create reproducible development and testing environments locally.

<img src="pods-ui.png" alt="Cloud Pods Web UI" title="Cloud Pods Web UI" width="800px" />

## Installation

You can save and load the persistent state of Cloud Pods, you can use the [Cloud Pods command-line interface (CLI)]({{< ref "pods-cli" >}}).
LocalStack provides a remote storage backend that can be used to store the state of your running application and share it with your team members.
You can interact with the Cloud Pods over the storage backend via the LocalStack Web Application.

Cloud Pods CLI is included in the [LocalStack CLI installation]({{< ref "/getting-started/installation/#localstack-cli" >}}), so there's no need for additional installations to begin using it.
If you're a licensed user, we suggest setting the `LOCALSTACK_AUTH_TOKEN` as an environment variable.
This enables you to access the complete range of LocalStack Cloud Pods features.

You can access the Cloud Pods CLI by running the `pod` command from your terminal.

{{< command >}}
$ localstack pod --help
Usage: localstack pod [OPTIONS] COMMAND [ARGS]...
<disable-copy>
  Manage the state of your instance via Cloud Pods.

Options:
  -h, --help  Show this message and exit.

Commands:
  delete    Delete a Cloud Pod
  inspect
  list      List all available Cloud Pods
  load
  remote    Manage cloud pod remotes
  save      Create a new Cloud Pod
  versions
</disable-copy>
{{< / command >}}

{{< callout >}}
These Cloud Pods are securely stored within an AWS storage backend, where each user or organization is allocated a dedicated and isolated S3 bucket.
The LocalStack Cloud Pods CLI utilizes secure S3 presigned URLs to directly interface with the S3 bucket, bypassing the need to transmit the state files through LocalStack Platform APIs.
{{< /callout >}}

## Getting started

This guide is designed for users new to Cloud Pods and assumes basic knowledge of the LocalStack CLI and our [`awslocal`](https://github.com/localstack/awscli-local) wrapper script.

Start your LocalStack container using your preferred method.
We will demonstrate how you can save a snapshot of your active LocalStack instance into your LocalStack account, and pull it to a running instance.

### Create AWS resources

You can use the `awslocal` CLI to create new AWS resources within your active LocalStack instance.
For example, you can create an S3 bucket and add data to it using the `awslocal` CLI:

{{< command >}}
$ awslocal s3 mb s3://test
$ echo "hello world" > /tmp/hello-world
$ awslocal s3 cp /tmp/hello-world s3://test/hello-world
$ awslocal s3 ls s3://test/
{{< / command >}}

### Save your Cloud Pod state

You can now save your Pod state using the `save` command, specifying the desired Cloud Pod name as the first argument.
This action will save the pod and register it with the LocalStack Web Application:

{{< command >}}
$ localstack pod save s3-test
<disable-copy>
Cloud Pod `s3-test` successfully created ✅
Version: 1
Remote: platform
Services: s3
</disable-copy>
{{< / command >}}

Optionally, you can include a message with the saved Cloud Pod using the `--message` flag.

You can access the list of available Cloud Pods for both you and your organization by utilizing the `list` command:

{{< command >}}
$ localstack pod list
<disable-copy>
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┓
┃ Name                         ┃ Max Version ┃ Last Change         ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━┩
│ s3-test                      │ 1           │ 2024-01-04 11:03:00 │
└──────────────────────────────┴─────────────┴─────────────────────┘
</disable-copy>
{{< / command >}}

With the `save` command you can create multiple versions of a Cloud Pod.
For instance, let us create a SQS queue and second version of `s3-test`.

{{< command >}}
$ awslocal sqs create-queue --queue-name test-queue

$ localstack pod save s3-test
<disable-copy>
Cloud Pod `s3-test` successfully created ✅
Version: 2
Remote: platform
Services: s3,sqs
</disable-copy>
{{< /command >}}

We can now use the command `versions` to list all the created version for a Cloud Pod.

{{< command >}}
$ localstack pod versions s3-test
<disable-copy>
┏━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━━━━┓
┃ Version ┃ Creation Date       ┃ LocalStack Version      ┃ Services ┃ Description ┃
┡━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━━━━┩
│ 1       │ 2024-01-04 11:03:00 │ 3.1.1.                  │ s3       │             │
│ 2       │ 2024-02-28 14:01:45 │ 3.1.1.                  │ s3,sqs   │             │
└─────────┴─────────────────────┴─────────────────────────┴──────────┴─────────────┘
</disable-copy>
{{< /command >}}

### Pull your Pod state

On a separate machine, start LocalStack while ensuring the Auth Token is properly configured.
Then, retrieve the previously created Cloud Pod by employing the `load` command, specifying the Cloud Pod name as the first argument:

{{< command >}}
$ localstack pod load s3-test
<disable-copy>
Cloud Pod s3-test successfully loaded
</disable-copy>
{{< / command >}}

You can examine the S3 buckets within the Cloud Pod:

{{< command >}}
$ awslocal s3 ls s3://test/
<disable-copy>
2022-10-04 22:33:54         12 hello-world
</disable-copy>
{{< / command >}}

You can also load a specific version by appending a version number to the pod name after a colon.
If not specified, the latest version will be loaded.
{{< command >}}
$ localstack pod load s3-test:1
<disable-copy>
Cloud Pod s3-test:1 successfully loaded
</disable-copy>
{{< / command >}}

After loading the Cloud Pod's content, you can use the `state inspect` command to observe the state of the running LocalStack instance.

{{< command >}}
$ localstack state inspect --format json
<disable-copy>
{
    "000000000000": {
        "S3": {
            "global": {
                "listBuckets": {
                    "Buckets": [
                        {
                            "Name": "test",
                            "CreationDate": "2023-10-03T07:19:31.000Z"
                        }
                    ],
                }
            }
        }
    }
}
</disable-copy>
{{< / command >}}

For comprehensive instructions, navigate to our [Command-Line Interface (CLI) Guide]({{< ref "pods-cli" >}}).
To access your Cloud Pods through the LocalStack Web Application, navigate to the [Cloud Pods browser](https://app.localstack.cloud/pods).

{{< callout >}}
Permission on Cloud Pods are assigned at organization level.
This means that every individual in the organization can view, load, and delete Cloud Pods created by other team members.
Similarly, everyone can save a new version of a Cloud Pod on top of a Pod originally created by someone else.
{{< /callout >}}

## Web Application

The LocalStack Web Application enables you to :

- Browse your Cloud Pods and access your version history.
- Export & import Cloud Pods to and from LocalStack instances.
- View Cloud Pods metadata, resources, regions, and version history.

### Browse Cloud Pods

[Cloud Pods Browser](https://app.localstack.cloud/pods) allows you to view, manage, and explore your Cloud Pods through the LocalStack Web Application.
With Cloud Pods, you can have individual or shared ownership of a snapshot of your LocalStack instance.

<img src="cloud-pods-browser.png" alt="LocalStack Web Application's Cloud Pods Browser outlining various saved Clod Pods" title="Cloud Pods Browser" width="900" />
<br><br>

The Cloud Pods Browser provides the following functionalities:

- **View Cloud Pods**: View all Cloud Pods saved by you or your organization.
- **View Versions**: View the version history of a Cloud Pod and access previous versions of specific Cloud Pods by clicking on the Cloud Pod's name.
- **View Cloud Pod Details**: View the details of a specific Cloud Pod version by clicking on the version.
- **View Cloud Pod storage**: View the organization storage usage and user storage usage on top of the Cloud Pods Browser.
- **Delete Cloud Pod**: Delete a Cloud Pod by selecting the Cloud Pod and navigating to the **Actions** button, followed by **Delete**.

### View Cloud Pods metadata

You can view Cloud Pods metadata by selecting any Cloud Pod in the  [Cloud Pods Browser](https://app.localstack.cloud/pods).
The metadata includes details such as:

- The user who created the Cloud Pod
- The creation timestamp
- The LocalStack version used to create the Cloud Pod
- The size of the Cloud Pod
- The service resources contained in the Cloud Pod

You can view detailed information within a Cloud Pod, including available resources, categorized services with configurations, and quick access to resource identifiers and endpoints—all without loading the Cloud Pod into your LocalStack runtime.

To save metadata with resource details in the Cloud Pod, ensure your LocalStack container is running and save the Cloud Pod with `ENABLE_POD_RESOURCES=1`.
Cloud Pods saved without this configuration enabled will not display granular details.

<img src="cloud-pod-details.png" alt="Cloud Pods details" title="Cloud Pods details" width="900" />
<br><br>

### Export & Import Cloud Pods

You can export and import your LocalStack infrastructure state as a Cloud Pod using the LocalStack Web Application.
This feature is particularly useful when you need to use a user-friendly interface to manage your Cloud Pods, without the need to interact with the CLI.

<img src="export-import-state-cloud-pod.png" alt="LocalStack Export/Import State Cloud Pod Mode" title="LocalStack Export/Import State Cloud Pod Mode" width="900" />

#### Export the State

To export the state, follow these steps:

1. Navigate to the **Cloud Pod** tab within the [Export/Import State](https://app.localstack.cloud/inst/default/state) page.
2. Create AWS resources locally as needed.
3. Enter the Pod name and toggle between the **New Pod** and **Existing Pod** options.
4. Enter the services to save resources for.
  By default, all available service resources are saved.
5. Click on **Create New Pod**.

A new Cloud Pod will be created and will be available for import into another LocalStack instance.
You can check out the list of available Cloud Pods in the [Cloud Pod](https://app.localstack.cloud/pods) page.

#### Import the State

To import the state, follow these steps:

1. Navigate to the **Cloud Pod** tab within the [Export/Import State](https://app.localstack.cloud/inst/default/state) page.
2. Choose the Cloud Pod from the drop-down list.
3. Click on **Load State From Pod**.

To confirm the successful injection of the container state, visit the respective [Resource Browser](https://app.localstack.cloud/inst/default/resources) for the services and verify the resources.

## Auto Loading Cloud Pods

In addition to loading Cloud Pods through the Command-Line Interface (CLI) or the Web Application, you can configure the automatic loading of one or more Cloud Pods upon the startup of the LocalStack container.

### Environmental variables

To automatically load a Cloud Pod at startup, utilize the `AUTO_LOAD_POD` [configuration variable]({{< ref "configuration" >}}).

`AUTO_LOAD_POD` can accept multiple Cloud Pod names separated by commas.
To autoload multiple Cloud Pods, such as `foo-pod` and `bar-pod`, use: `AUTO_LOAD_POD=foo-pod,bar-pod`.
The order of Cloud Pods in `AUTO_LOAD_POD` dictates their loading sequence.
When autoloading multiple Cloud Pods, later pods might overwrite the state of earlier ones if they share the same service, account, and region.

{{< tabpane lang="bash" >}}
{{< tab header="LocalStack CLI" lang="bash" >}}
AUTO_LOAD_POD=foo-pod localstack start
{{< /tab >}}
{{< tab header="Docker Compose" lang="yaml" >}}
services:
  localstack:
    container_name: "localstack-main"
    image: localstack/localstack-pro
    ports:
      - "127.0.0.1:4566:4566"
      - "127.0.0.1:4510-4559:4510-4559"
    environment:
      - LOCALSTACK_AUTH_TOKEN=${LOCALSTACK_AUTH_TOKEN:?}
      - DEBUG=1
      - AUTO_LOAD_POD=foo-pod,bar-pod
    volumes:
      - "./volume:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
{{< /tab >}}
{{< tab header="Docker" lang="bash" >}}
docker run \
  -e LOCALSTACK_AUTH_TOKEN=${LOCALSTACK_AUTH_TOKEN:?} \
  -e AUTO_LOAD_POD=foo-pod \
  -v ./volume:/var/lib/localstack \
  -p 4566:4566 \
  localstack/localstack-pro
{{< /tab >}}
{{< /tabpane >}}

### Configuration file

LocalStack allows for the use of configuration files to automatically load Cloud Pods during startup.

Within the container, LocalStack searches through the `/etc/localstack/init-pod.d` directory for two file types: `zip` files created using the `localstack state export` command, and `txt` files, where each line represents the name of a Cloud Pod.

Take the following example of a project layout:

```bash
.
├── docker-compose.yml
└── init-pods.d
    ├── pod-list.txt
    └── my-state.pod.zip
```

The `pod-list.txt` contains the following:

```text
foo-pod
bar-pod
```

LocalStack, upon mounting `init-pods.d` to the appropriate location, will sequentially load `foo-pod` and `bar-pod` as listed in `pod-list.txt`, and then proceed to load `my-state.pod.zip`.

The docker compose file for correctly mounting `init-pods.d` will look like:

```yaml
services:
  localstack:
    container_name: "localstack-main"
    image: localstack/localstack-pro
    ports:
      - "127.0.0.1:4566:4566"
      - "127.0.0.1:4510-4559:4510-4559"
    environment:
      - LOCALSTACK_AUTH_TOKEN=${LOCALSTACK_AUTH_TOKEN:?}
      - DEBUG=1
    volumes:
      - "./volume:/var/lib/localstack"
      - "./init-pods.d:/etc/localstack/init-pods.d"
```

## Remotes

A remote is the location where Cloud Pods are stored.
By default, Cloud Pod artifacts are stored in the LocalStack platform.
However, if your organization's data regulations or sovereignty requirements prohibit storing Cloud Pod assets in a remote storage infrastructure, you have the option to persist Cloud Pods in an on-premises storage location under your complete control.

LocalStack provides two types of alternative remotes:

- S3 bucket remote storage.
- [ORAS](https://oras.land/) (OCI Registry as Storage) remote storage.

Cloud Pods command-line interface (CLI) allows you to create, delete, and list remotes.

{{< command >}}
$ localstack pod remote --help
<disable-copy>
Usage: localstack pod remote [OPTIONS] COMMAND [ARGS]...

  Manage cloud pod remotes

Options:
  -h, --help  Show this message and exit.

Commands:
  add     Add a remote
  delete  Delete a remote
  list    List the available remotes
</disable-copy>
{{< / command >}}

### S3 bucket remote storage

The S3 remote enables you to store Cloud Pod assets in an existing S3 bucket within an actual AWS account.
The initial step is to export the necessary AWS credentials within the terminal session.

```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
```

A possible option is to obtain credentials via [AWS SSO CLI](https://github.com/synfinatic/aws-sso-cli).

Next, we establish a new remote specifically designed for an S3 bucket.
By running the following command, we create a remote named `s3-storage-aws` responsible for storing Cloud Pod artifacts in an S3 bucket called `ls-pods-bucket-test`.

The `access_key_id` and `secret_access_key` placeholders ensure the correct transmission of AWS credentials to the container.

{{< command >}}
$ localstack pod remote add s3-storage-aws 's3://ls-pods-bucket-test/?access_key_id={access_key_id}&secret_access_key={secret_access_key}'
{{< / command >}}

Lastly, you can utilize the standard `pod` CLI command to generate a new Cloud Pod that points to the previously established remote.

{{< command >}}
$ localstack pod save my-pod s3-storage-aws
{{< / command >}}

Once the command has been executed, you can confirm the presence of Cloud Pod artifacts in the S3 bucket by simply running:

```bash
aws s3 ls s3://ls-pods-bucket-test
2023-09-27 13:50:10      83650 localstack-pod-my-pod-state-1.zip
2023-09-27 13:50:11      85103 localstack-pod-my-pod-version-1.zip
```

You can use the `pod load` command to load the same pod that was previously saved in this remote:

{{< command >}}
$ localstack pod load my-pod s3-storage-aws
{{< / command >}}

Similarly, you can list the Cloud Pods on this specific remote with the `pod list` command:

{{< command >}}
$ localstack pod list s3-storage-aws
{{< / command >}}

{{< callout >}}
Full S3 remotes support is available in the CLI from version 3.2.0.
If you experience any difficulties, update your [LocalStack CLI]({{< ref "/getting-started/installation/#updating" >}}).
{{< /callout >}}

### ORAS remote storage

The ORAS remote enables users to store Cloud Pods in OCI-compatible registries like Docker Hub, Nexus, or ECS registries.
ORAS stands for "OCI Registry as Service," and you can find additional information about this standard [on the official website](https://oras.land/).

For example, let's illustrate how you can utilize Docker Hub to store and retrieve Cloud Pods.

To begin, you must configure the new remote using the LocalStack CLI.
You'll need to export two essential environment variables, `ORAS_USERNAME` and `ORAS_PASSWORD`, which are necessary for authenticating with Docker Hub.

```bash
export ORAS_USERNAME=docker_hub_id
export ORAS_PASSWORD=ILoveLocalStack1!
```

You can now use the CLI to create a new remote called `oras-remote`.

{{< command >}}
$ localstack pod remote add oras-remote 'oras://{oras_username}:{oras_password}@registry.hub.docker.com/<docker_hub_id>'
{{< / command >}}

Lastly, you can store a pod using the newly configured remote, where `my-pod` represents the Cloud Pod's name, and `oras-remote` is the remote's name.

{{< command >}}
$ localstack pod save my-pod oras-remote
{{< / command >}}

Likewise, you can execute the reverse operation to load a Cloud Pod from `oras-remote` using the following command:

{{< command >}}
$ localstack pod load my-pod oras-remote
{{< / command >}}

### Auto Load with remotes

LocalStack also supports the auto load of a Cloud Pod from registered remotes.
The configuration is similar to what we just described.
In particular you could simply add the remote name to the text files inside the `init-pods.d`, as follows:

```text
foo-pod,bar-remote
```

With such a configuration, the `foo-pod` Cloud Pod will be loaded from the `bar-remote` remote.
To properly configure the remote, you need to provide the needed environment variables when starting the LocalStack container.
For instance, a S3 remote needs a `AWS_ACCESS_KEY` and a `AWS_SECRET_ACCESS_KEY`, as follows:

```yaml
services:
  localstack:
    container_name: "localstack-main"
    image: localstack/localstack-pro
    ports:
      - "127.0.0.1:4566:4566"
      - "127.0.0.1:4510-4559:4510-4559"
    environment:
      - LOCALSTACK_AUTH_TOKEN=${LOCALSTACK_AUTH_TOKEN:?}
      - DEBUG=1
      - AWS_ACCESS_KEY_ID:...
      - AWS_SECRET_ACCESS_KEY:...
    volumes:
      - "./volume:/var/lib/localstack"
      - "./init-pods.d:/etc/localstack/init-pods.d"
```

{{< callout >}}
The Auto Load from remote feature does not automatically configure the remote.
This needs to be done with the `localstack pod remote add ...` command.
This commands creates a configuration file for the remote in the [LocalStack volume directory]({{< ref "/references/filesystem/#localstack-volume" >}}).
{{< /callout >}}

## End-to-End Encryption (Enterprise)

Cloud Pods artifacts are stored in S3 buckets when using the LocalStack platform as the storage remote.
By default, Amazon S3 encrypts all objects before saving them on disks, while the opposite operation happens at download time.
This ensures encryption **at rest** for Cloud Pods.

When this is not enough, LocalStack also offers end-to-end encryption for enterprise customers as a preview feature.
To activate this feature, make sure to start LocalStack with the `POD_ENCRYPTION` environment variable set to 1.

The next step is to generate a passphrase used to encrypt and decrypt the Cloud Pods' artifacts.
We advise to create a strong passphrase by using the `openssl` utility, e,g.:

{{< command >}}
$ openssl rand --base64 32
# 3X03eU5pgoejObUR+Y8I4QjbjeGEKjDcmVFd0FU5pCg=

{{< / command >}}

Users should treat the generated passphrase as a secret and they are responsible for securely sharing it within the organization.
The generated secret can now be provided as an option to the `save` command when creating an encrypted Cloud Pod.

{{< command >}}
$ localstack pod save my-secret-pod --secret 3X03eU5pgoejObUR+Y8I4QjbjeGEKjDcmVFd0FU5pCg=
{{< / command >}}

Loading an encrypted Cloud Pod would require a similar `load` command:

{{< command >}}
$ localstack pod load my-secret-pod --secret 3X03eU5pgoejObUR+Y8I4QjbjeGEKjDcmVFd0FU5pCg=
{{< / command >}}

### Encryption with PGP keys

We also offer the option of using PGP keys to encrypt and decrypt Cloud Pods.
The process is the following:

- Customers would have to export both their private and public keys into two files, `private.pgp` and `public.pgp` respectively.
- These files need to be mounted in a specific `pods.keys.d` folder when starting LocalStack, i.e., `localstack start -v $PWD/pods.keys.d:/etc/localstack/pods.keys.d`.
- The `secret` option passed to the `save` and `load` command corresponds to the passphrase needed to import the private key into the LocalStack runtime.

### Limitations

- Both browsing the Cloud Pod content via the UI and loading Cloud Pods into ephemeral instances are currently not supported for encrypted Cloud Pods.
- It is not possible to have both encrypted and non-encrypted versions for a Cloud Pod.
  Encryption is set at the moment of the creation and it cannot be changed.

### Miscellaneous

Unless explicitly specified, all Cloud Pods commands default to targeting the LocalStack Platform as the storage remote.
It's important to note that the CLI must be authenticated correctly with our Platform.

Custom remote configurations are stored within the [LocalStack volume directory]({{< ref "/references/filesystem/#localstack-volume-directory" >}}) and are managed by the LocalStack container.
Consequently, when sharing Cloud Pods among your team using a custom remote, each team member must define the identical remote configuration.
Once added, a remote persists even after LocalStack restarts.

## State Merging

Cloud Pods offers various strategies for integrating states into your LocalStack container.
The available strategies are:

- `overwrite`: This strategy clears the existing state and loads the new state from the Cloud Pod, completely resetting the LocalStack state.
- `account-region-merge` (**default**): This strategy merges services based on account and region pairs.
  It attempts to combine states from both the current state and the Cloud Pod for the same account and region.
- `service-merge`: This strategy merges services at the account-region level, provided there's no overlap in resources.
  It prioritizes the loaded resources when merging.

### LocalStack CLI

To activate merge strategies, set `--strategy <strategy>` when loading a Cloud Pod using the LocalStack CLI.
For instance, to load a Cloud Pod named `test-pod-s3-sqs` with the `service-merge` strategy, run the following command:

{{< command >}}
$ localstack pod load test-pod-s3-sqs --strategy service-merge
{{< / command >}}

### LocalStack Web Application

To activate merge strategies, navigate to the **Cloud Pods** tab on the [Export/Import State page](https://app.localstack.cloud/inst/default/state).
Enter the name of the Cloud Pod, select the version, choose the strategy from a dropdown, and click **Load State from Pod**.

<img src="merge-strategy-web-app.png" alt="Merge Strategy Web UI" title="Merge Strategy Web UI" width="800px" />

### Example scenario

Let us take the image below as example.
The two non overlapping account/region pairs (`0123456789/us-east-1` for the Cloud Pod and `0123456789/us-east-2` for the runtime) will be both present in the resulting state.
For `0123456789/eu-central-1` however, we encounter a conflict, since both the Cloud Pod and the container hold a SQS state.
With the `account-region-merge` strategy, the one from the Cloud Pod will be preserved.

<img src="merge-strategies.png" alt="Merge Strategies" title="Merge Strategies" width="800px" />
<br><br>

On the other hand, in the `service-merge` strategy, the SQS resulting state will have 2 distinct queues if the queue from the Cloud Pod and the one in the container are distinct, i.e., do not have the same ARN.
In case of an ARN conflict, only one queue, the one from the Cloud Pod, will be present in the result.

### Dry Run

To preview the changes that would occur when loading a Cloud Pod, you can use the `--dry-run` flag.
The result will depend on the selected merge strategy.
The result will be displayed in the console, and no changes will be made to the LocalStack state.

```bash
This load operation will modify the runtime state as follows:

──────────────────────────── sns ────────────────────────────
+ 2 resources added.
~ 1 resources modified.

──────────────────────── cognito-idp ────────────────────────
+ 1 resources added.
~ 0 resources modified.

──────────────────────────── sqs ────────────────────────────
+ 1 resources added.
~ 1 resources modified.
```

## Cloud Pods & Persistence

[Persistence]({{< ref "persistence" >}}) ensures that the service state persists across container restarts.
You can enable persistence via a LocalStack config flag `PERSISTENCE=1` to restore your local resources, in case you’re stopping and re-starting the LocalStack instance on the same machine.

In contrast, Cloud Pods provide more detailed control over your state.
Rather than just restoring a state during LocalStack restarts, Cloud Pods enable you to capture snapshots of your local instance using the `save` command and inject these snapshots into a running instance using the `load` command, all without needing to perform a full restart.

### Current Limitations

Cloud Pods (and state management in general), come with a few limitation.
In particular, Cloud Pods states might not be correctly restored if the LocalStack version used to create the pod and the target one differ.
We detect version miss-matches when using the `pod load` and prompt a confirmation message to the user.

{{< command >}}
$ localstack pod load old-pod
<disable-copy>
This Cloud Pod was created with LocalStack 2.1.0.
but you are running LocalStack 3.2.1.
Cloud Pods might be incompatible across different LocalStack versions.
Loading a Cloud Pod with mismatching version might lead to a corrupted state of the emulator.
Do you want to continue? [y/N]:
</disable-copy>
{{< / command >}}

We are working to extend Cloud Pods support to all AWS services emulated in LocalStack.
However, state management might not yet work reliably for every service.

## Troubleshooting

### Unable to obtain Auth Token

When you try to save a Cloud Pod and see the error in LocalStack logs like this:

```bash
localstack.cli.exceptions.CLIError: Failed to create Cloud Pod sample-pod ❌ - Unable to obtain auth token (code 401) - please log in again.
```

It would be good to check if you have outdated authentication credentials (bearer token from a previous LocalStack login) in the `remotes.yaml` file for cloud pods.
You have two options to fix this:

1. Run another `localstack auth login` command.
2. Find the `remotes.yaml` file in the `<localstack_volume>` directory on your machine and delete the file, or at least remove the `"default"` entry from it.

Additionally, if there is a `~/.localstack/auth.json` file in your home directory, delete it as well if it still exists.

### License not found

When you try to save a Cloud Pod and see the `license.not_found` error in LocalStack logs like this:

```bash
lsmulti-localstack  | 2024-03-15T13:06:16.358  WARN --- [functhread31] l.p.remotes.remotes        : Failed to register pod sample-pod: {"error": true, "message": "licensing.license.not_found"}
```

To fix this, clear the LocalStack cache directory and restart the LocalStack instance before trying to save the Cloud Pod again.
You can find the cache directories at:

- `/Users/localstack/Library/Caches/localstack`
- `/Users/localstack/Library/Caches/localstack-cli`

Adjust the path based on your operating system.

### SSL Certificate verification failed

If you get an SSL certificate verification error while trying to save a Cloud Pod, as shown below:

```bash
An error occurred while checking remote management for pod "cloud-pod-product-app": "MyHTTPSConnectionPool(host='api.localstack.cloud', port=443): Max retries exceeded with url: /v1/cloudpods/cloud-pod-product-app (Caused by SSLError(SSLCertVerificationError(1, "[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: Hostname mismatch, certificate is not valid for 'api.localstack.cloud'. (_ssl.c:1006)")))"
```

Check if your machine's clock is set incorrectly or if the certificate store is outdated.
This error can also occur if you use `localstack` as `LOCALSTACK_HOST`.
In this case, the DNS incorrectly resolves `api.localstack.cloud` to `localhost`, causing a certificate mismatch.

### `InvalidAccessKeyId` during `CreateBucket` API call

You may see this error:

```bash
An error occurred (InvalidAccessKeyId) when calling the CreateBucket operation: The AWS Access Key Id you provided does not exist in our records.
```

This usually means you're using temporary AWS credentials, but `AWS_SESSION_TOKEN` is missing from the environment.

To fix this, ensure that you export the `AWS_SESSION_TOKEN` environment variable:

{{< command >}}
$ export AWS_SESSION_TOKEN=...
{{< / command >}}

Then, add `session_token` to the `pod remote add` URL:

{{< command >}}
$ localstack pod remote add s3-storage-aws 's3://ls-pods-bucket-test/?access_key_id={access_key_id}&secret_access_key={secret_access_key}&session_token={session_token}'
{{< / command >}}

If you're still unable to connect to the S3 bucket, add the bucket hostname to the upstream DNS resolution list:

```bash
DNS_NAME_PATTERNS_TO_RESOLVE_UPSTREAM=ls-pods-bucket-test.s3.amazonaws.com/
```

For more details, see the [Skip LocalStack DNS Resolution]({{< ref "/user-guide/tools/dns-server/#skip-localstack-dns-resolution" >}}) section.
