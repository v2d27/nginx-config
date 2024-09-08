## Sample config NGINX Load Balancing for Kubernetes
![Project Image](https://nginx.org/nginx.png)

In this sample, we use NGINX as a load balancer to manage and distribute incoming traffic across multiple instances of our application running in a Kubernetes cluster. NGINX is a powerful, high-performance web server and reverse proxy that is widely used for load balancing.

### Why Use NGINX for Load Balancing?

- **High Performance**: NGINX is known for its speed and efficiency, handling a large number of simultaneous connections with low resource consumption.
- **Flexible Configuration**: NGINX provides a rich set of features and configurations for load balancing, including various algorithms like round-robin, least connections, and IP hash.
- **Robust Features**: NGINX supports advanced features such as SSL termination, caching, and custom health checks.

### How It Works

1. **Deployment**: Deploy NGINX as a load balancer within your Kubernetes cluster. This typically involves creating a Kubernetes Deployment or DaemonSet to manage the NGINX pods.
2. **Service Configuration**: Configure a Kubernetes Service of type `LoadBalancer` or `NodePort` to expose the NGINX deployment and allow external traffic to reach it.
3. **Ingress Resource**: Use a Kubernetes Ingress resource to define routing rules for your application. NGINX will route incoming requests to the appropriate backend services based on these rules.
4. **Scaling**: As your application scales, NGINX will automatically distribute traffic across the available instances, ensuring balanced load and improved availability.

## Table of Contents
* **NGINX Open Source** supports five load‑balancing methods:
  - [Round Robin](#Round-Robin)
  - [Weighted Round Robin](#Weighted-Round-Robin)
  - [IP Hash](#IP-Hash)
  - [Least Connection](#Least-Connection)
  - [Generic Hash](#Generic-Hash)
* And **NGINX Plus** adds two more methods:
  - [Least Response Time](#Least-Response-Time)
  - [Random](#random) 

## Round Robin
The Round Robin algorithm is a simple static load balancing approach in which requests are distributed across the servers in a sequential or rotational manner. It is easy to implement but it doesn’t consider the load already on a server so there is a risk that one of the servers receives a lot of requests and becomes overloaded.

![roundrobin](https://github.com/v2d27/nginx-config/raw/main/images/Round-Robin.webp)
Requests are distributed evenly across the servers, with server weights taken into consideration. This method is used by default (there is no directive for enabling it):

```nginx
#install nginx Round-Robin Load Balancing
upstream k8s_cluster {
    server 192.168.1.18:31000;
    server 192.168.1.19:31000;
    server 192.168.1.20:31000;
}
server {
    listen 80;
    location / {
        proxy_pass http://k8s_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    error_log /var/log/nginx/k8s_loadbalancer_error.log;
    access_log /var/log/nginx/k8s_loadbalancer_access.log;
}
```

## Weighted Round Robin
The Weighted Round Robin algorithm is also a static load balancing approach which is much similar to the round-robin technique. The only difference is, that each of the resources in a list is provided a weighted score. Depending on the weighted score the request is distributed to these servers. 

![weighted](https://github.com/v2d27/nginx-config/raw/main/images/Weighted%20Round-Robin%20Load%20Balancing.webp)

By default, NGINX distributes requests among the servers in the group according to their weights using the Round Robin method. The weight parameter to the server directive sets the weight of a server; the `default` is `1`:

```nginx
#install nginx Weighted Round-Robin Load Balancing
upstream k8s_cluster {
    server 192.168.1.18:31000 weight=10; # weighted example
    server 192.168.1.19:31000 weight=15; # weighted example
    server 192.168.1.20:31000 weight=10; # weighted example
}
server {
    listen 80;

    location / {
        proxy_pass http://k8s_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    error_log /var/log/nginx/k8s_loadbalancer_error.log;
    access_log /var/log/nginx/k8s_loadbalancer_access.log;
}
```


## IP Hash
The Source IP Hash Load Balancing Algorithm is a method used in network load balancing to distribute incoming requests among a set of servers based on the hash value of the source IP address. This algorithm aims to ensure that requests originating from the same source IP address are consistently directed to the same server.

![iphash](https://github.com/v2d27/nginx-config/raw/main/images/Source-IP-hash.webp)

The server to which a request is sent is determined from the client IP address. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available.
```nginx
#install nginx IP Hash Load Balancing
upstream k8s_cluster {
    ip_hash;
    server 192.168.1.18:31000;
    server 192.168.1.19:31000;
    server 192.168.1.20:31000;
}
server {
    listen 80;

    location / {
        proxy_pass http://k8s_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    error_log /var/log/nginx/k8s_loadbalancer_error.log;
    access_log /var/log/nginx/k8s_loadbalancer_access.log;
}
```

## Least Connection
The Least Connections algorithm is a dynamic load balancing approach that assigns new requests to the server with the fewest active connections. The idea is to distribute incoming workloads in a way that minimizes the current load on each server, aiming for a balanced distribution of connections across all available resources. 

- To do this load balancer needs to do some additional computing to identify the server with the least number of connections. 
- This may be a little bit costlier compared to the round-robin method but the evaluation is based on the current load on the server. 

![leastconnection](https://github.com/v2d27/nginx-config/raw/main/images/Least-Connection.webp)

A request is sent to the server with the least number of active connections, again with server weights taken into consideration:
```nginx
#install nginx Least Connections Load Balancing
upstream k8s_cluster {
    least_conn;
    server 192.168.1.18:31000;
    server 192.168.1.19:31000;
    server 192.168.1.20:31000;
}
server {
    listen 80;
    location / {
        proxy_pass http://k8s_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    error_log /var/log/nginx/k8s_loadbalancer_error.log;
    access_log /var/log/nginx/k8s_loadbalancer_access.log;
}
```

## Generic Hash
Generic Hash – The server to which a request is sent is determined from a user‑defined key which can be a text string, variable, or a combination. For example, the key may be a paired source IP address and port, or a URI as in this example:

```nginx
upstream backend {
    hash $request_uri consistent;
    server backend1.example.com;
    server backend2.example.com;
}
```

The optional consistent parameter to the hash directive enables ketama consistent‑hash load balancing. Requests are evenly distributed across all upstream servers based on the user‑defined hashed key value. If an upstream server is added to or removed from an upstream group, only a few keys are remapped which minimizes cache misses in the case of load‑balancing cache servers or other applications that accumulate state.

## Least Response Time
The Least Response Time method is a dynamic load balancing approach that aims to minimize response times by directing new requests to the server with the quickest response time. 

It considers the historical performance of servers to make decisions about where to route incoming requests, optimizing for faster processing.
The dynamic aspect comes from the continuous monitoring of server response times and the adaptive nature of the algorithm to route incoming requests to the server with the historically lowest response time

![leastrespondse](https://github.com/v2d27/nginx-config/raw/main/images/Least-Response.webp)

**Least Time (NGINX Plus only)** – For each request, NGINX Plus selects the server with the lowest average latency and the lowest number of active connections, where the lowest average latency is calculated based on which of the following parameters to the `least_time` directive is included:
`header` – Time to receive the first byte from the server
`last_byte` – Time to receive the full response from the server
`last_byte inflight` – Time to receive the full response from the server, taking into account incomplete requests

```nginx
upstream backend {
    least_time header;
    server backend1.example.com;
    server backend2.example.com;
}
```

## Random
**Random** – Each request will be passed to a randomly selected server. If the two parameter is specified, first, NGINX randomly selects two servers taking into account server weights, and then chooses one of these servers using the specified method:

`least_conn` – The least number of active connections
`least_time=header` (NGINX Plus) – The least average time to receive the response header from the server ($upstream_header_time)
`least_time=last_byte` (NGINX Plus) – The least average time to receive the full response from the server ($upstream_response_time)
```nginx
upstream backend {
    random two least_time=last_byte;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
    server backend4.example.com;
}
```
The Random load balancing method should be used for distributed environments where multiple load balancers are passing requests to the same set of backends. For environments where the load balancer has a full view of all requests, use other load balancing methods, such as round robin, least connections and least time.

***Note: When configuring any method other than Round Robin, put the corresponding directive (`hash`, `ip_hash`, `least_conn`, `least_time`, or `random`) above the list of server directives in the upstream {} block.***


## Source
- [Using nginx as load balancer](https://nginx.org/en/docs/http/load_balancing.html)
- [Load Balancing Algorithms](https://www.geeksforgeeks.org/load-balancing-algorithms/)
- [HTTP Load Balancing NGINX](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
