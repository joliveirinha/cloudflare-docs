---
title: Create a locally-managed tunnel (CLI)
pcx_content_type: how-to
weight: 2
layout: single
---

# Set up a tunnel locally

Follow this step-by-step guide to get your first tunnel up and running using the CLI.

## Prerequisites

Before you start, make sure you:

- [Add a website to Cloudflare](/fundamentals/setup/account-setup/add-site/).
- [Change your domain nameservers to Cloudflare](/dns/zone-setups/full-setup/setup/).

## 1. Download and install `cloudflared`

{{<tabs labels="Windows | macOS | Linux | Build from source">}}
{{<tab label="windows" no-code="true">}}

1. Download `cloudflared` on your machine. Visit the [downloads](/cloudflare-one/connections/connect-networks/downloads/) page to find the right package for your OS.

2. Rename the executable to `cloudflared.exe`

3. In PowerShell, change directory to your Downloads folder and run `.\cloudflared.exe --version`. It should output the version of `cloudflared`. Note that `cloudflared.exe` could be `cloudflared-windows-amd64.exe` or `cloudflared-windows-386.exe` if you have not renamed it.

```bash
PS C:\Users\Administrator\Downloads\cloudflared-stable-windows-amd64> .\cloudflared.exe --version
```

{{</tab>}}
{{<tab label="macos" no-code="true">}}
 
To download and install `cloudflared`:

```sh
$ brew install cloudflare/cloudflare/cloudflared
```

Alternatively, you can [download the latest Darwin amd64 release](/cloudflare-one/connections/connect-networks/downloads/) directly.
 
{{</tab>}}
{{<tab label="linux" no-code="true">}}

**Debian and Ubuntu APT**

Use the apt package manager to install `cloudflared` on compatible machines.

{{<render file="tunnel/_cloudflared-debian-install.md">}}

**RHEL RPM**

Use the rpm package manager to install `cloudflared` on compatible machines.

1. Add Cloudflare's repository:

```sh
$ curl -fsSL https://pkg.cloudflare.com/cloudflared-ascii.repo | sudo tee /etc/yum.repos.d/cloudflared.repo
```

2. Update repositories and install cloudflared:

```sh
$ sudo yum update && sudo yum install cloudflared
```

**Arch Linux**

`cloudflared` is in the Arch Linux [`community` repository](https://wiki.archlinux.org/title/official_repositories#community).
Use `pacman` to install `cloudflared` on compatible machines.

```sh
$ pacman -Syu cloudflared
```

**Other**

Alternatively you can download the `cloudflared` binary or the linux packages to your machine and install manually. Visit the [downloads](/cloudflare-one/connections/connect-networks/downloads/) page to find the right package for your OS.

{{</tab>}}
{{<tab label="build from source" no-code="true">}}

To build the latest version of `cloudflared` from source:

```sh
$ git clone https://github.com/cloudflare/cloudflared.git
$ cd cloudflared
$ make cloudflared
$ go install github.com/cloudflare/cloudflared/cmd/cloudflared
```

Depending on where you installed `cloudflared`, you can move it to a known path as well.

```sh
$ mv /root/cloudflared/cloudflared /usr/bin/cloudflared
```

{{</tab>}}
{{</tabs>}}

## 2. Authenticate `cloudflared`

```sh
$ cloudflared tunnel login
```

Running this command will:

- Open a browser window and prompt you to log in to your Cloudflare account. After logging in to your account, select your hostname.
- Generate an account certificate, the [cert.pem file](/cloudflare-one/connections/connect-networks/get-started/tunnel-useful-terms/#certpem), in the [default `cloudflared` directory](/cloudflare-one/connections/connect-networks/get-started/tunnel-useful-terms/#default-cloudflared-directory).

## 3. Create a tunnel and give it a name

```sh
$ cloudflared tunnel create <NAME>
```

Running this command will:

- Create a tunnel by establishing a persistent relationship between the [name you provide](/cloudflare-one/connections/connect-networks/get-started/tunnel-useful-terms/#tunnel-name) and a [UUID](/cloudflare-one/connections/connect-networks/get-started/tunnel-useful-terms/#tunnel-uuid) for your tunnel. At this point, no connection is active within the tunnel yet.
- Generate a [tunnel credentials file](/cloudflare-one/connections/connect-networks/get-started/tunnel-useful-terms/#credentials-file) in the [default `cloudflared` directory](/cloudflare-one/connections/connect-networks/get-started/tunnel-useful-terms/#default-cloudflared-directory).
- Create a subdomain of `.cfargotunnel.com`.

From the output of the command, take note of the tunnel’s UUID and the path to your tunnel’s credentials file.

Confirm that the tunnel has been successfully created by running:

```sh
$ cloudflared tunnel list
```

## 4. Create a configuration file

1. In your `.cloudflared` directory, create a [`config.yml` file](/cloudflare-one/connections/connect-networks/configure-tunnels/local-management/configuration-file/)  using any text editor. This file will configure the tunnel to route traffic from a given origin to the hostname of your choice.

2. Add the following fields to the file:

**If you are connecting an application:**

```yml
---
filename: config.yml
---
url: http://localhost:8000
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
```

**If you are connecting a private network:**

```yml
---
filename: config.yml
---
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
warp-routing:
    enabled: true
```

3. Confirm that the configuration file has been successfully created by running:

    ```sh
    $ cat config.yml
    ```

## 5. Start routing traffic

1. Now assign a `CNAME` record that points traffic to your tunnel subdomain:

    - If you are connecting an application, route the service to a [public hostname](/cloudflare-one/connections/connect-networks/routing-to-tunnel/):

    ```sh
    $ cloudflared tunnel route dns <UUID or NAME> <hostname>
    ```

    - If you are connecting a [private network](/cloudflare-one/connections/connect-networks/private-net/), route an IP address or CIDR through the tunnel:

    ```sh
    $ cloudflared tunnel route ip add <IP/CIDR> <UUID or NAME>
    ```

2. Confirm that the route has been successfully established:

    ```sh
    $ cloudflared tunnel route ip show
    ```

## 6. Run the tunnel

Run the tunnel to proxy incoming traffic from the tunnel to any number of services running locally on your origin.

```sh
$ cloudflared tunnel run <UUID or NAME>
```

If your configuration file has a custom name or is not in the `.cloudflared` directory, add the `--config` flag and specify the path.

```sh
$ cloudflared tunnel --config /path/your-config-file.yml run <UUID or NAME>
```

{{<Aside>}}

Cloudflare Tunnel can install itself as a system service on Linux and Windows and as a launch agent on macOS. For more information, refer to [run as a service](/cloudflare-one/connections/connect-networks/configure-tunnels/local-management/as-a-service/).

{{</Aside>}}

## 7. Check the tunnel

Your tunnel configuration is complete! If you want to get information on the tunnel you just created, you can run:

```sh
$ cloudflared tunnel info <UUID or NAME>
```

You can now [route traffic](/cloudflare-one/connections/connect-networks/routing-to-tunnel/) to your tunnel using Cloudflare DNS or [determine who can reach your tunnel](/cloudflare-one/policies/access/) with Cloudflare Access.
