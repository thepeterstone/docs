---
title: Proxy Manual Install
tags: [proxies, best practice]
sidebar: doc_sidebar
permalink: proxies_manual_install.html
summary: Install a Wavefront proxy or Telegraf agent from the command line.
---
In some environments, it's necessary to perform a proxy manual installation, for example, to accomodate network restrictions. This page gives some guidance. [Install and Manage Wavefront Proxies](proxies_installing.html#test-a-proxy) explains how to test the proxy.

{% include tip.html content="Because the exact steps depend on your environment, this page can't give details for each use case." %}


## Prerequisite: Proxy Host Network Connectivity

Before you begin the installation process, test connectivity between the host being configured and your Wavefront service. You can test connectivity from the proxy host to the Wavefront server using curl.

Run this test before installing the proxy, and again after installing and configuring the proxy.

1. Find the values for Wavefront server and token. Here's an easy way to do it:
   1. Select **Integrations** from the task bar.
   2. Select **Linux Host** and click the **Setup** tab.
2. Run the following command:
   ```
   curl -v https://your-server.wavefront.com/api/daemon/test?token=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   ```

Here is an example of the expected return when you use the `-v` parameter (without `-v` only HTTP(S) errors are reported):
```
* About to connect() to myhost.wavefront.com port 443 (#0)
*   Trying NN.NN.NNN.NNN...
* Connected to myhost.wavefront.com (NN.NN.NNN.NNN) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* SSL connection using TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
* Server certificate:
* 	subject: CN=*.wavefront.com,O="VMware, Inc",L=Palo Alto,ST=California,C=US
* 	start date: <date>
* 	expire date: <date>
* 	common name: *.wavefront.com
* 	issuer: <issuer details>
> GET /api/daemon/test?token=<mytoken> HTTP/1.1
> User-Agent: curl/7.29.0
> Host: myhost.wavefront.com
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx
< Date: Wed, 09 Jan 2019 20:41:45 GMT
< Transfer-Encoding: chunked
< Connection: keep-alive
< X-Upstream: 10.15.N.NNN>:NNNNN
< X-Wavefront-Cluster: /services-<services>
< X-Frame-Options: SAMEORIGIN
<
* Connection #0 to host myhost.wavefront.com left intact
```


## Proxy Package Install - Full Network Access

Follow these steps to install a proxy on a host with full network access (incoming and outgoing connections).


### Prerequisites

* **JRE**. Before you download the proxy file, consider which JRE is to be used.
  * If JRE 8/9/10/11 is already installed on the machine and is in the execution path, then proxy will use the installed version, otherwise it will attempt to download the JRE.
  * If you would like to use a specific JRE version that is **not** in the execution path, add
  ```
  export PROXY_JAVA_HOME=<path_to_JRE_of_your_choice>
  ```
  to `/etc/sysconfig/wavefront-proxy` (create the file if it does not exist)

* **Required Information**. You need the following information to start the proxy.

  {% include note.html content="To find the values for server and token, you can select **Integrations** from the task bar, select **Linux Host**, and select the **Setup** Tab."%}

  <table style="width: 100%;">
  <thead>
  <tr><th width="20%">Parameter</th><th width="60%">Description</th><th width="20%">Example</th></tr>
  </thead>
  <tr>
  <td markdown="span">**server**</td>
  <td>URL of the Wavefront server you log in to.  </td>
  <td>https://proxy1.acme.org/api/ </td>
  </tr>
  <tr>
  <td markdown="span">**token**</td>
  <td markdown="span">API token. See <a href="users_account_managing.html#generate-an-api-token"Generate an API Token</a></td>
  <td>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx </td>
  </tr>
  <tr>
  <td markdown="span">**hostname**</td>
  <td markdown="span">Name of the host the proxy will run on (alphanumeric and periods are allowed). The hostname is used to tag data internal to the proxy, such as JVM statistics, per-proxy point rates, and so on. </td>
  <td>cust42ProxyHost</td>
  </tr>
  </tbody>
  </table>

### Download, Configure, and Start the Proxy

1. Download the proxy `.rpm` or `.deb` file from [packagecloud.io/wavefront/proxy](http://packagecloud.io/wavefront/proxy).
2. Run `sudo rpm -U <name_of_file.rpm>` or `sudo dpkg -i <name_of_file.deb>`.
3. Specify required parameters in the config file or when running `autoconf-wavefront-proxy.sh` (discussed next)

**Option 1: Use a Config File**

If you want to customize the proxy using the [proxy configuration file](proxies_configuring.html), follow these steps:

1. Open the file, available [in these locations](proxies_configuring.html#paths) by default.
2. A a minimum, specify values for the following parameters:
  * server= &lt;Wavefront server&gt;
  * hostname= &lt;proxy host&gt;
  * token=&lt;API token&gt;
  You can set additional configuration properties, for example, to require SSL connections (discussed below).
3 . Start the Wavefront proxy service:
   ```
   sudo service wavefront-proxy start
   ```

**Option 2: Run the autoconf Script**

When you run `bin/autoconf-wavefront-proxy.sh`, you're prompted for the required settings.

After the interactive configuration is complete:
* The Wavefront proxy configuration at `/etc/wavefront/wavefront-proxy/wavefront.conf` is updated with the input that you provided.
* The `wavefront-proxy` service starts.


## Proxy Install -- Limited Network Access

Some Wavefront customers run the proxy on a host with limited network access.

### Prerequisites

- **Networking:** The minimum requirement is an outbound HTTPS connection to the Wavefront service so the proxy can send metrics to the Wavefront service.
  * The proxy uses port 2878 by default. It's possible to specify custom ports the following parameters:
    - `pushListenerPorts`
    - `deltaCountersAggregationListenerPorts`
    - `traceListenerPorts`
    - `histogramListenerPorts`
  * You can use an [HTTP proxy](proxies_manual_install.html#connecting-to-wavefront-through-an-http-proxy) for the connection.

- **JRE:** The Wavefront proxy is a Java jar file and requires a JRE - for example, openjdk11.
  - If JRE 8/9/10/11 is already installed on the machine and is in the execution path, then proxy will use the installed version, otherwise it will attempt to download the JRE.
  - If you would like to use a specific JRE version that is not in the execution path, add
  ```
  export PROXY_JAVA_HOME=<path_to_JRE_of_your_choice>
  ```
  to `/etc/sysconfig/wavefront-proxy` (create the file if it does not exist)


### Install and Configure the Proxy (Limited Network Access)

To install the proxy in an environment with limited network access:

1. Install the .rpm or .deb file.
2. (Optional) Update the settings, either by editing the configuration file or by running the `autoconf` script, .
4. You may need to update the Wavefront control file  `/etc/init.d/wavefront.proxy` to the following settings:

```
   desc=${DESC:-Wavefront Proxy}
   user="wavefront"
   wavefront_dir="/opt/wavefront"
   proxy_dir=${PROXY_DIR:-$wavefront_dir/wavefront-proxy}
   config_dir=${CONFIG_DIR:-/etc/wavefront/wavefront-proxy}
   proxy_jre_dir="$proxy_dir/proxy-jre"                <-- set to location of currently installed JRE
   export JAVA_HOME=${PROXY_JAVA_HOME:-$proxy_jre_dir} <-- set to location of currently installed JRE
```

## Proxy Install -- Incoming TLS/SSL

The Wavefront proxy can accept incoming TCP and HTTP requests on port 2878 or custom ports. You can secure proxy connections by accepting only connections with a certificate and key:

1. Specify which port to use:
   * Metrics: `pushListenerPorts`
   * Histograms: `histogramDistListenerPorts`
   * Traces: `traceListenerPorts`
   See [Set the Listener Port](proxies_installing.html#set-the-listener-port-for-metrics-histograms-and-traces)
2. Specify the `tlsPorts`, `privateKeyPath`, and `privateCertPath` parameters. You can specify those parameters in the configuration file or by running `bin/autoconf-wavefront-proxy.sh`.
   <table style="width: 100%;">
   <tbody>
   <thead>
   <tr><th width="20%">Parameter</th><th width="80%">Description</th></tr></thead>
   <tr>
   <td markdown="span">**tlsPorts**</td>
   <td markdown="span">Comma-separated list of ports to be used for incoming TLS/SSL connections. </td>
   </tr>
   <tr>
   <td markdown="span">**privateKeyPath**</td>
   <td markdown="span">Path to PKCS#8 private key file in PEM format. Incoming TLS/SSL connections access this private key. </td>
   </tr>
   <tr>
   <td markdown="span">**privateCertPath**</td>
   <td markdown="span">Path to X.509 certifivate chain file in PEM format. Incoming TLS/SSL connections access this certificate. </td>
   </tr>
   </tbody>
   </table>


## Connecting to Wavefront Through an HTTP Proxy

The Wavefront proxy initiates an HTTPS connection to the Wavefront service. The connection is made over the default https port 443. Connection parameters are configured in `/etc/wavefront/wavefront-proxy/wavefront.conf`.

If direct connection is not available, you can use a HTTP proxy to connect to the Wavefront service

{% include note.html content="The Wavefront proxy does not use the `http_proxy` environment variables. You must specify the information in `wavefront.conf`. See [Proxy Configuration File Paths](proxies_configuring.html#proxy-configuration-file-paths)" %}


By default, the HTTP proxy section is commented out. Uncomment the section and specify the values required by your HTTP proxy, for example:


```
## The following settings are used to connect to Wavefront servers through a HTTP proxy:
#proxyHost=localhost
#proxyPort=8080
## Optional: if http proxy requires authentication
#proxyUser=proxy_user
#proxyPassword=proxy_password
#
```
You can then follow Wavefront proxy setup steps.


## Install Proxies on Multiple Linux Hosts

Ansible is an open-source automation engine that automates software provisioning, configuration and management, and application deployment. The Wavefront Ansible role installs and configures the Wavefront proxy, which allows you to automate Wavefront proxy installation on multiple Linux hosts.

{% include note.html content="In most cases, you install only one or two proxies in your environment. You don't need a proxy for each host you collect data from. See [Proxy Deployment Options](proxies.html#proxy-deployment-options)." %}

For details, see [Wavefront Ansible Role Setup](ansible.html#wavefront-ansible-role-setup).
<!---Vasily, the ansible doc page is generated from the integration. Is it correct? --->


<a name="docker"></a>

## Configure a Proxy in a Container

You can use the in-product Docker with cAdvisor or Kubernetes integration if you want to set up a proxy in a container. You can then customize that proxy.

### Proxy Versions for Containers
For containers, the proxy image version is determined by the `image` property in the configuration file. You can set this to `image: wavefronthq/proxy:latest`, or specify a proxy version explicitly.
The proxies are not stateful. Your configuration is managed in your `yaml` file. It's safe to use  `proxy:latest` -- we ensure that proxies are backward compatible.

### Restrict Memory Usage for the Container

To restrict memory usage of the container using Docker, you need to add a `JAVA_HEAP_USAGE` environment variable and restrict memory using the `-m` or `--memory` options for the docker `run` command.  The container memory contraint should be at least 350mb larger than the JAVA_HEAP_USAGE environment variable.

To restrict a container's memory usage to 2g with Docker run:

```docker run -d --name wavefront-proxy ... -e JAVA_HEAP_USAGE="1650m" -m 2g ...```

To limit memory usage of the container in Kubernetes use the `resources.limits.memory` property of a container definition. See the [Kubernetes doc](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/).

### Customize Proxy Settings for Docker

When you run a Wavefront proxy inside a Docker container, you can tweak proxy configuration settings that are properties in the `wavefront.conf` file directly from the Docker `run` command. You use the WAVEFRONT_PROXY_ARGS environment variable and pass in the property name as a long form argument, preceded by `--`.

For example, add `-e WAVEFRONT_PROXY_ARGS="--pushRateLimit 1000"` to your docker `run` command to specify a rate limit of 1000 pps for the proxy.

See the [Wavefront Proxy configuration file](https://github.com/wavefrontHQ/java/blob/master/pkg/etc/wavefront/wavefront-proxy/wavefront.conf.default) for a full list.

### Log Customization for Containers

You can customize logging by mounting a customized `log4j2.xml` file. Here's an example for Docker:

```
--mount type=bind, src=<absolute_path>/log4j2.xml, dst=/etc/wavefront/wavefront-proxy/log4j2.xml
```
See **Logging** above for additional background.

## Install Telegraf Manually

If the system has network access, follow our [instructions for installing Telegraf](https://packagecloud.io/wavefront/telegraf/install)

You can use .deb, .rpm, node, python, and gem for installing from the network.

If you're in an environment with restricted network access:

1. Download the appropriate Telegraf package [from InfluxData](https://portal.influxdata.com/downloads/) and install as directed.
2. Create a file called `10-wavefront.conf` in `/etc/telegraf/telegraf.d` and enter the following snippet:
```
[[outputs.wavefront]]
  host = "WAVEFRONT_PROXY_ADDRESS"
  port = 2878
  metric_separator = "."
  source_override = ["hostname", "agent_host", "node_host"]
  convert_paths = true
```
3. Start the Telegraf agent:

```
   sudo service telegraf start
```
