# consul-minikubes

A script to create a number of Minikube clusters to test Consul.

## Prerequisites

- Minikube 1.30.1
- VirtualBox 6.1

## Usage

Start a set of three clusters.

```shell
bash start.sh
```

You can use the `-n` flag to pass in
any number of clusters.

```shell
bash start.sh -n 5
```

> Note: This script will peer all clusters to each other for exploration.
> In production, you will only want to peer clusters with services that
> need to access each other for least privilege.

Clean up the clusters and networking in VirtualBox.

```shell
bash clean.sh
```

## Explore

If you want to log into any of the Consul clusters, use the URLs and
ACL tokens for each Consul datacenter in `.consul.state`.

Deploy an example set of services to dc1, dc2, and dc3.

```shell
bash example.sh
```

This creates a 3-tier application across multiple Consul / Kubernetes clusters.

```shell
web  ───────►  application  ────────►  database
dc1            dc2                     dc3
```

To test the example, access the `web` service in `dc1`.

```shell
$ curl http://$(kubectl --context dc1 get service web -o=jsonpath='{.status.loadBalancer.ingress[0].ip}'):9090

{
  "name": "web",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.244.0.11"
  ],
  "start_time": "2023-04-19T16:26:53.922278",
  "end_time": "2023-04-19T16:26:53.946828",
  "duration": "24.55018ms",
  "body": "Response from web",
  "upstream_calls": {
    "http://application.virtual.dc2.consul": {
      "name": "application",
      "uri": "http://application.virtual.dc2.consul",
      "type": "HTTP",
      "ip_addresses": [
        "10.244.0.11"
      ],
      "start_time": "2023-04-19T16:26:53.934568",
      "end_time": "2023-04-19T16:26:53.946412",
      "duration": "11.84421ms",
      "headers": {
        "Content-Length": "856",
        "Content-Type": "text/plain; charset=utf-8",
        "Date": "Wed, 19 Apr 2023 16:26:53 GMT"
      },
      "body": "Response from application",
      "upstream_calls": {
        "http://database.virtual.dc3.consul": {
          "name": "database",
          "uri": "http://database.virtual.dc3.consul",
          "type": "HTTP",
          "ip_addresses": [
            "10.244.0.11"
          ],
          "start_time": "2023-04-19T16:26:53.944459",
          "end_time": "2023-04-19T16:26:53.944846",
          "duration": "387.918µs",
          "headers": {
            "Content-Length": "269",
            "Content-Type": "text/plain; charset=utf-8",
            "Date": "Wed, 19 Apr 2023 16:26:53 GMT"
          },
          "body": "Response from database",
          "code": 200
        }
      },
      "code": 200
    }
  },
  "code": 200
}
```

## Debug

If you have trouble with networking, review `.minikubes.state` to match
the network name in Virtualbox with the Minikube profile.