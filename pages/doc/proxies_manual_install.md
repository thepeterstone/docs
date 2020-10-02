---
title: Proxy Manual Install
tags: [proxies, best practice]
sidebar: doc_sidebar
permalink: proxies_manual_install.html
summary: Install a Wavefront proxy or Telegraf agent from the command line.
---
In some environments, it's necessary to perform a proxy manual installation, for example, to accomodate network restrictions. This page gives some guidance. You can perform additional customization using [proxy configuration properties](proxies_configuring.html#general-proxy-properties-and-examples).

{% include tip.html content="Because the exact steps depend on your environment, this page can't give details for each use case. [Advanced Proxy Configuration](proxies_configuring.html) lists all configuration parameters with examples." %}


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


### Step 1: Download the Proxy

Download the proxy file:

1. Download the proxy `.rpm` or `.deb` file from [packagecloud.io/wavefront/proxy](http://packagecloud.io/wavefront/proxy).
2. Run `sudo rpm -U <name_of_file.rpm>` or `sudo dpkg -i <name_of_file.deb>`.
    {% include note.html content="<br/>If no Java JRE is in the path, this command installs JRE locally under `/opt/wavefront/wavefront-proxy/proxy-jre`." %}

<!---Vasily, is this step belabouring the obvious? We could cut it
### Step 2: Determine Configuration Information

You need the following information to customize the settings.

{% include note.html content="To find the values for server and token, you can select **Integrations** from the task bar, select **Linux Host**, and select the **Setup** Tab."%}

<table style="width: 100%;">
<tbody>
<thead>
<tr><th width="20%">Parameter</th><th width="60%">Description</th><th width="20%">Example</th></tr>
</thead>
<tr>
<td markdown="span">**server**</td>
<td>URL of the Wavefront server you log in to.  </td>
<td>https://try.wavefront.com/api/ </td>
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
<tr>
<td markdown="span">**tlsPorts**</td>
<td markdown="span">Comma-separated list of ports to be used for incoming HTTPS connections. </td>
</tr>
<tr>
<td markdown="span">**privateKeyPath**</td>
<td markdown="span">Path to PKCS#8 private key file in PEM format. Incoming HTTPS connections access this private key. </td>
</tr>
<tr>
<td markdown="span">**privateCertPath**</td>
<td markdown="span">Path to X.509 certifivate chain file in PEM format. Incoming HTTPS connections access this certificate. </td>
</tr>
</tbody>
</table>
--->

### Step 3: Configure the Proxy

You can configure the proxy by editing the config file or by specifying input when running `autoconf-wavefront-proxy.sh`.

#### Option 1: Use a Config File

If you edit the [proxy configuration file](proxies_configuring.html) before you start the proxy, you can specify additional configuration.

1. Find the configuration file, available [in these locations](proxies_configuring.html#paths) by default.
2. Specify values for the following parameters:
  * server= &lt;Wavefront server&gt;
  * hostname= &lt;proxy host&gt;
  * token=&lt;API token&gt;
  You can set additional configuration properties, for example, to require SSL connections (discussed below) or to enable graphite format metrics.
3 . Start the Wavefront proxy service:
   ```
   sudo service wavefront-proxy start
   ```

#### Option 2: Run the autoconf Script

When you run `bin/autoconf-wavefront-proxy.sh`, you're prompted for the required settings.

After the interactive configuration is complete:
* The Wavefront proxy configuration at `/etc/wavefront/wavefront-proxy/wavefront.conf` is updated with the input that you provided.
* The `wavefront-proxy` service is started.


## Proxy Install -- Limited Network Access

Some Wavefront customers want to run the proxy on a host with limited network access.

### Prerequisites

- **Networking:** The minimum requirement is an outbound HTTPS connection to the Wavefront service so the proxy can send metrics to the Wavefront service.
  * For metrics, the proxy uses port 2878 by default. Customize the port with the  `pushListenerPorts` parameter and configure [additional proxy ports](proxies_installing.html#configuring-proxy-ports-for-metrics-histograms-and-traces) for histograms and traces as needed.

  * You can use an [HTTP proxy](proxies_manual_install.html#connecting-to-wavefront-through-an-http-proxy) for the connection.

- **JRE:** The Wavefront proxy is a Java jar file and requires a JRE - for example, openjdk8.  If the JRE is in the execution path you should be able to install the .rpm or .deb file as above.

<!---Vasily: Above, Is that at a minimum openjdk8?--->

### Installation and Configuration Steps

Installation and configuration is similar to environments with full network access but might require additional work.

1. Make sure all prerequisites are met, including an open outgoing HTTPS connection to the Wavefront service and JRE.
2. Install the .rpm or .deb file.
3. Update the settings, either by editing the configuration file or by running the `autoconf` script, as explained above.
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

The Wavefront proxy can accept incoming TCP and HTTP requests on the port specified by `pushListenerPorts`. You can secure proxy connections by accepting only connections with a certificate and key:

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




## Testing Your Installation

After you have started the proxy you just configured, you can verify its status from the Wavefront UI or with curl commands.

### Testing From the UI
To check your proxy from the UI:
1. Log in to your Wavefront instance from a browser.
2. From the task bar, select **Browse > Proxies** to view a list of all proxies.
   If the list is long, type the proxy name (defined in `hostname=` in  `wavefront.conf`) to locate the proxy by name.

### Testing Using curl

You can test your proxy using `curl`. Documentation for the following curl commands can be found directly on your Wavefront server at `https://<your-server.wavefront.com>/api-docs/ui/#!/Proxy/getAllProxy`.

You can run the commands [directly from the API documentation](https://www.wavefront.com/wavefront-rest-api/). This is less error prone than copy/paste of the token.

For this task, you first you get the list of proxies for your Wavefront service, then you display information for just the proxy you installed.

1. Get the list of all proxies for your Wavefront server:
   ```
   curl -X GET --header "Accept: application/json" --header "Authorization: Bearer xxxxxxxxx-<your api token>-xxxxxxxxxxxx" "https://<yourserver.wavefront.com/api/v2/proxy?offset=0&limit=100"
   ```
   This command returns a JSON formated list of all proxies.

2. Get the proxy ID. You can search the output using the proxy name configured in `wavefront.conf`, or find the proxy ID in the UI.

3. Return information for only this proxy:
   ```
   curl -X GET --header "Accept: application/json" --header "Authorization: Bearer xxxxxxxxx-<your api token>-xxxxxxxxxxxx" "https://<yourserver>.wavefront.com/api/v2/proxy/443e5771-67c8-40fc-a0e2-674675d1e0a6"
   ```

Sample output for single proxy:
<!---Vasily, is the (a) useful and (b) up to date?--->

```
{
    "status": {
        "result": "OK",
        "message": "",
        "code": 200
    },
    "response": {
        "customerStatus": "ACTIVE",
        "version": "4.34",
        "status": "ACTIVE",
        "customerId": "mike",
        "inTrash": false,
        "hostname": "mikeKubeH",
        "id": "443e5771-67c8-40fc-a0e2-674675d1e0a6",
        "lastCheckInTime": 1547069052859,
        "timeDrift": -728,
        "bytesLeftForBuffer": 7624290304,
        "bytesPerMinuteForBuffer": 31817,
        "localQueueSize": 0,
        "sshAgent": false,
        "ephemeral": false,
        "deleted": false,
        "statusCause": "",
        "name": "Proxy on mikeKubeH"
    }
}
```

## Connecting to Wavefront Through an HTTP Proxy

The Wavefront proxy initiates an HTTPS connection to the Wavefront service. The connection is made over the default https port 443. Connection parameters are configured in `/etc/wavefront/wavefront-proxy/wavefront.conf`.

You can instead configure the HTTP proxy setting within `wavefront.conf`.
<!--Vasily, should that say: "You can instead use an HTTP proxy for the connection between the Wavefront proxy and the Wavefront service." It's a little vague right now--->

{% include note.html content="The Wavefront proxy does not use the `http_proxy` environment variables. You must specify the information in `wavefront.conf`." %}

### Customize wavefront.conf for HTTP Connections

By default, the HTTP proxy section is commented out. Uncomment the section and specify the values required by your HTTP proxy, for example:

<!---Vasily, do we need to tell users where to find wavefront.conf--->

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

### Set http_proxy Environment Variables

<!---Vasily: Is this useful? Wouldn't it be better to just use the config file? --->

Here are examples of setting the `http_proxy` environment variables. These variables are not used by the proxy itself.

```
Set these variables to configure Linux proxy server settings for the command-line tools:

$ export http_proxy="http://PROXY_SERVER:PORT"
$ export https_proxy="https://PROXY_SERVER:PORT"
$ export ftp_proxy="http://PROXY_SERVER:PORT"
If a proxy server requires authentication, set the proxy variables as follows:

$ export http_proxy="http://USER:PASSWORD@PROXY_SERVER:PORT"
$ export https_proxy="https://USER:PASSWORD@PROXY_SERVER:PORT"
$ export ftp_proxy="http://USER:PASSWORD@PROXY_SERVER:PORT"
```


## Install Telegraf Manually

If the system has network access, follow our [instructions for installing Telegraf](https://packagecloud.io/wavefront/telegraf/install)

We include instructions for using .deb, .rpm, node, python, and gem for installing from the network.

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
