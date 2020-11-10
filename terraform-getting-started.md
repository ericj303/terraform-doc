# Getting Started with Terraform
Terraform is the most popular tool for defining and provisioning infrastructure as code (IaC).  In this tutorial you'll use Terraform to provision a NGINX web server running in a Docker container.

## Prerequisites

- Terraform **version 0.12.0** or later
- [Docker](https://www.docker.com/)

## Install Terraform
To download and install Terraform for your OS and architecture, go to [Terraform.io](https://www.terraform.io/downloads.html).

With Terraform installed, you can setup your project.

## Setup project
Create a new directory.

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

Next, create a file for your Terraform configuration.

```shell
$ touch main.tf
```

Paste the following code into the file.

```hcl
provider "docker" {
    host = "unix:///var/run/docker.sock"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

## Initialize Terraform
Use the `init` command to initialize Terraform. The Docker provider will be installed. 

```shell
$ terraform init
Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "docker" (terraform-providers/docker) 2.7.2...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.docker: version = "~> 2.7"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

If it ran successfully, you'll see the message `Terraform has been successfully initialized`.  You can now provision the resources using the `apply` command.

## Provision infrastructure
When you run the `terraform apply` command, a plan is generated showing you the potential resource changes.
Terraform waits for your approval before proceeding.

Type `yes` at the confirmation prompt to apply the plan.

```shell
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + dns              = (known after apply)
      + dns_opts         = (known after apply)
      + entrypoint       = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = (known after apply)
      + log_opts         = (known after apply)
      + logs             = false
      + must_run         = true
      + name             = "training"
      + network_data     = (known after apply)
      + read_only        = false
      + restart          = "no"
      + rm               = false
      + shm_size         = (known after apply)
      + start            = true
      + user             = (known after apply)
      + working_dir      = (known after apply)

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id     = (known after apply)
      + latest = (known after apply)
      + name   = "nginx:latest"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```

Enter `yes`.  The command will take up to a few minutes to run and then display a message indicating that the resources were created.

```shell
docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 4s [id=sha256:c39a868aad02a383c7e490e0fc4a5b0217f667f2de764bc2755e315a5adf64a1nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 0s [id=99045024a1afc7e32b8bc9a5e813281bb80241a29ad7c0f3c9b2a5774dcd0c4f]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

To verify the provisioning, run the `docker ps` command to see the running container, and `curl http://localhost` to see the NGINX server's landing page.

```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                NAMES
99045024a1af        c39a868aad02        "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   training
$ curl http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

## Destroy infrastructure
Finally, destroy the infrastructure.  Confirm the action by entering `yes`.

```shell
$ terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:c39a868aad02a383c7e490e0fc4a5b0217f667f2de764bc2755e315a5adf64a1nginx:latest]
docker_container.nginx: Refreshing state... [id=99045024a1afc7e32b8bc9a5e813281bb80241a29ad7c0f3c9b2a5774dcd0c4f]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach            = false -> null
      - command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      - entrypoint        = [
          - "/docker-entrypoint.sh",
        ] -> null
      - gateway           = "172.17.0.1" -> null
      - group_add         = [] -> null
      - hostname          = "99045024a1af" -> null
      - id                = "99045024a1afc7e32b8bc9a5e813281bb80241a29ad7c0f3c9b2a5774dcd0c4f" -> null
      - image             = "sha256:c39a868aad02a383c7e490e0fc4a5b0217f667f2de764bc2755e315a5adf64a1" -> null
      - ip_address        = "172.17.0.2" -> null
      - ip_prefix_length  = 16 -> null
      - ipc_mode          = "private" -> null
      - links             = [] -> null
      - log_driver        = "json-file" -> null
      - log_opts          = {} -> null
      - logs              = false -> null
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      - must_run          = true -> null
      - name              = "training" -> null
      - network_data      = [
          - {
              - gateway          = "172.17.0.1"
              - ip_address       = "172.17.0.2"
              - ip_prefix_length = 16
              - network_name     = "bridge"
            },
        ] -> null
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      - read_only         = false -> null
      - restart           = "no" -> null
      - rm                = false -> null
      - shm_size          = 64 -> null
      - start             = true -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }

      - ulimit {
          - hard = 4096 -> null
          - name = "nofile" -> null
          - soft = 1024 -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id     = "sha256:c39a868aad02a383c7e490e0fc4a5b0217f667f2de764bc2755e315a5adf64a1nginx:latest" -> null
      - latest = "sha256:c39a868aad02a383c7e490e0fc4a5b0217f667f2de764bc2755e315a5adf64a1" -> null
      - name   = "nginx:latest" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: 
  ```

  After you enter yes, Terraform will destroy all resources.

  ```shell

docker_container.nginx: Destroying... [id=99045024a1afc7e32b8bc9a5e813281bb80241a29ad7c0f3c9b2a5774dcd0c4f]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:c39a868aad02a383c7e490e0fc4a5b0217f667f2de764bc2755e315a5adf64a1nginx:latest]
docker_image.nginx: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
```

Confirm the running NGINX Docker container has been removed by running `docker ps` and `curl http://localhost` again.

```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
$ curl http://localhost
curl: (7) Failed to connect to localhost port 80: Connection refused
```


## Next steps

In this tutorial, you used Terraform to provision and then destroy a NGINX web server running in Docker.

To learn more about the Terraform configuration language, please visit the following documentation pages.

[Terraform Tutorials](https://learn.hashicorp.com/terraform)

[Resource Documentation](https://www.terraform.io/docs/configuration/resources.html)

[Input Variable Documentation](https://www.terraform.io/docs/configuration/variables.html)

[Terraform Registry Providers](https://registry.terraform.io/browse/providers)
