# [KONG][website-url] :heavy_plus_sign: [DCOS Deployment](https://dcos.io/docs/1.9/)

[![Website][website-badge]][website-url]
[![Documentation][documentation-badge]][documentation-url]
[![Mailing List][mailing-list-badge]][mailing-list-url]
[![Gitter Badge][gitter-badge]][gitter-url]

[![][kong-logo]][website-url]

Provision Kong on Mesosphere DC/OS cluster using following steps.

Kong can be provisioned on a Mesosphere DC/OS cluster using following
steps:

The following steps use AWS for provisioning the DC/OS cluster and assumes you 
have basic knowledge of [DC/OS](https://dcos.io/docs/1.9/), 
[Marathon](https://mesosphere.github.io/marathon/), 
[VIPs](https://dcos.io/docs/1.9/networking/load-balancing-vips/virtual-ip-addresses/),
and [Marathon-LB](https://dcos.io/docs/1.9/networking/marathon-lb/).

1. **Initial setup**

    Download or clone the following repo:

    ```bash
    $ git clone git@github.com:Mashape/kong-dist-dcos.git
    $ cd kong-dist-dcos
    ```

    Skip to step 3 if you have already provisioned a DC/OS cluster.

2. **Deploy a DC/OS cluster**

    Following the [DC/OS AWS documentation](https://dcos.io/docs/1.9/installing/cloud/aws/),
    deploy a DC/OS cluster on which Kong will be provisioned
    
    Once your cluster is ready, Kong can be deployed using the
    [DC/OS CLI](https://dcos.io/docs/1.9/cli/) or the
    [DC/OS GUI](https://dcos.io/docs/1.9/gui/).

3. **Deploy Marathon-LB**

    We will use [Marathon-LB](https://dcos.io/docs/1.9/networking/marathon-lb/)
    for load balancing external traffic to cluster and
    [VIPs](https://dcos.io/docs/1.9/networking/load-balancing-vips/virtual-ip-addresses/)
    for load balancing internal traffic. Using the package `marathon-lb` deploy
    the Marathon-LB:

    ```bash
    $ dcos package install marathon-lb
    ```

4. **Deploy a Kong-supported database**

    Before deploying Kong, you need to provision a Cassandra or PostgreSQL
    instance.

    For Cassandra, use the `cassandra` package to deploy 3 nodes of Cassandra
    in the DC/OS cluster:

    ```bash
    $ dcos package install cassandra
    ```

    For PostgreSQL, use the `postgres.json` file from this repo to deploy
    a PostgreSQL instance in the cluster:

    ```bash
    $ dcos marathon app add postgres.json
    ```

5. **Deploy Kong**

    Using the `kong_<postgres|cassandra>.json` file from the kong-dist-dcos
    repo, deploy Kong in the cluster. Template registers Kong's `proxy` and
    `admin` ports on Marathon-LB as service ports `10001` and `10002`
    respectively:

    ```bash
    $ dcos marathon app add kong_<postgres|cassandra>.json
    ```

6. **Verify your deployments**

    If you followed the [AWS documentation](https://dcos.io/docs/1.8/administration/installing/cloud/aws/)
    to create the cluster, then you have to expose your Kong service ports
    on the public ELB to access the Kong services externally.
    
    You can also log into the public slave agent and test Kong by making the
    following requests: 

    ```bash
    $ curl marathon-lb.marathon.mesos:10001
    $ curl marathon-lb.marathon.mesos:10002
    ```

7. **Deploy an upstream server**

    For this demo, we created an app which returns `Hello world` on port `8080`.
    Using the `my_app.json` file from the kong-dist-dcos repo, deploy the app in
    the cluster which will act as a backend server to process requests received
    from Kong:

    ```bash
    $ dcos marathon app add my_app.json
    ```

8. **Using Kong**
    
    Create an API on Kong:

    ```bash
    $ curl -i -X POST marathon-lb.marathon.mesos:10002/apis \
    --data "name=myapp" \
    --data "hosts=myapp.com" \
    --data "upstream_url=http://myapp.marathon.l4lb.thisdcos.directory:8080"
    HTTP/1.1 201 Created
    ...

    ```

    Make a request to the API:

    ```bash
    $ curl -i -X GET marathon-lb.marathon.mesos:10001 \
    --header "Host:myapp.com"
    HTTP/1.1 200 OK
    ...

    Hello world
    ```

    Quickly learn how to use Kong with the 
    [5-minute Quickstart](https://getkong.org//docs/latest/getting-started/quickstart).

## Benchmarking Kong
  
  We prepared a simple benchmark by comparing performance of Nginx and
  Kong (with Postgres) running on DC/OS cluster. We used
  [Jmeter](http://jmeter.apache.org/usermanual/build-web-test-plan.html) to
  genrate sample requests and collect the performace data. We ran a Kong
  instance with one plugin `basic-auth` and a Nginx instance proxying request to
  host `mockbin.com`. Both Kong and Nginx instance assingned 1 cpu and 1 gb ram.
  We ran the test plan using 25 threads with 400 requests in loop.

### Mesurements

| Proxy   | Samples| Average| Min  | Max   | Std. Dev. | Error %  | Throughput | KB/sec | Avg. Bytes |
|---------|--------|--------|------|-------|-----------|----------|------------|--------|------------|
| Nginx   | 10000  | 202    | 192  | 1000  | 28.34     | 0.00%    | 109.6      | 184.83 | 1727.2     |
| Kong    | 10000  | 207    | 193  | 8574  | 109.46    | 0.00%    | 101.3      | 209.14 | 2113.4     |

## Enterprise Support

Support, Demo, Training, API Certifications and Consulting available 
at http://getkong.org/enterprise.

[kong-logo]: http://i.imgur.com/4jyQQAZ.png
[website-url]: https://getkong.org/
[website-badge]: https://img.shields.io/badge/GETKong.org-Learn%20More-43bf58.svg
[documentation-url]: https://getkong.org/docs/
[documentation-badge]: https://img.shields.io/badge/Documentation-Read%20Online-green.svg
[gitter-url]: https://gitter.im/Mashape/kong
[gitter-badge]: https://img.shields.io/badge/Gitter-Join%20Chat-blue.svg
[mailing-list-badge]: https://img.shields.io/badge/Email-Join%20Mailing%20List-blue.svg
[mailing-list-url]: https://groups.google.com/forum/#!forum/konglayer

