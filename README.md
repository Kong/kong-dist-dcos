# [KONG][website-url] :heavy_plus_sign: [DCOS Deployment](https://docs.mesosphere.com/1.8/overview/)

[![Website][website-badge]][website-url]
[![Documentation][documentation-badge]][documentation-url]
[![Mailing List][mailing-list-badge]][mailing-list-url]
[![Gitter Badge][gitter-badge]][gitter-url]

[![][kong-logo]][website-url]

Provision Kong on Mesosphere DC/OS cluster using following steps.

Following guide use AWS for provisioning the DC/OS Cluster and assumes you have basic knowledge of DC/OS, Marathon and Marathon-lb. You can spawn Cluster on any supported Platform or on a local machine 


1. **Initial setup**:

	## Deploy a cluster

	  Follow the DC/OS [AWS documentation](https://dcos.io/docs/1.8/administration/installing/cloud/aws/)

	Once you have cluster ready, Application can deployed using DC/OS [cli](https://docs.mesosphere.com/1.8/usage/cli/) or DCOS [GUI](https://docs.mesosphere.com/1.8/usage/webinterface/). 	  

2. **Deploy Marathon-lb**

	Download or clone the repo

    ```bash
    $ git clone git@github.com:Mashape/kong-dist-mesos.git
    $ cd kong-dist-mesos
    ```

    Deploy Marathon-lb for internal and external Service discovery support
	
    ```bash
    $ dcos package install marathon-lb
    $ dcos package install --options=marathon-lb-internal.json marathon-lb
    ```
3. **Deploy Kong supported Database**
	
    ```bash
    $ dcos marathon app add postgres.json
    ```

4. **Deploy Kong**
	
    ```bash
    $ dcos marathon app add kong.json
    ```

5. **Using Kong:**
	
	If you used [AWS documentation](https://dcos.io/docs/1.8/administration/installing/cloud/aws/) to create the cluster then you have to expose Kong service ports on Public ELB to access the kong Services externally. You can also log into the Public Slave agent and curl Kong services.  

    ```bash
    $ curl marathon-lb.marathon.mesos:10001
    $ curl marathon-lb.marathon.mesos:10002
    ```

    Quickly learn how to use Kong with the [5-minute Quickstart](/docs/latest/getting-started/quickstart).

## Benchmarking Kong
  
  We prepared a simple benchmark by comparing performance of Nginx and Kong running on DC/OS cluster. We used [Jmeter](http://jmeter.apache.org/usermanual/build-web-test-plan.html) to genrate sample requests and collect the performace data. We ran a Kong instance with one plugin `basic-auth` and a Nginx instance proxying request to
  host `mockbin.com`. Both Kong and Nginx instance assingned 1 cpu and 1 gb ram. We ran the test plan using 25 threads with 400 requests in loop.

### Mesurements

| Proxy   | Samples| Average| Min  | Max   | Std. Dev. | Error %  | Throughput | KB/sec | Avg. Bytes |
|---------|--------|--------|------|-------|-----------|----------|------------|--------|------------|
| Nginx   | 10000  | 202    | 192  | 1000  | 28.34     | 0.00%    | 109.6      | 184.83 | 1727.2     |
| Kong    | 10000  | 207    | 193  | 8574  | 109.46    | 0.00%    | 101.3      | 209.14 | 2113.4     |

## Enterprise Support

Support, Demo, Training, API Certifications and Consulting available at http://getkong.org/enterprise.

[kong-logo]: http://i.imgur.com/4jyQQAZ.png
[website-url]: https://getkong.org/
[website-badge]: https://img.shields.io/badge/GETKong.org-Learn%20More-43bf58.svg
[documentation-url]: https://getkong.org/docs/
[documentation-badge]: https://img.shields.io/badge/Documentation-Read%20Online-green.svg
[gitter-url]: https://gitter.im/Mashape/kong
[gitter-badge]: https://img.shields.io/badge/Gitter-Join%20Chat-blue.svg
[mailing-list-badge]: https://img.shields.io/badge/Email-Join%20Mailing%20List-blue.svg
[mailing-list-url]: https://groups.google.com/forum/#!forum/konglayer

