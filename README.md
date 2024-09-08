## NGINX Load Balancing for Kubernetes
![Project Image](https://nginx.org/nginx.png)

In this project, we use NGINX as a load balancer to manage and distribute incoming traffic across multiple instances of our application running in a Kubernetes cluster. NGINX is a powerful, high-performance web server and reverse proxy that is widely used for load balancing.

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
  **NGINX Open Source** supports four load‑balancing methods:
  - [Round Robin](#Round Robin)
  - [IP Hash](#IP Hash)
  - [Least Connection Method](#Least Connection Method)
  - [Generic Hash](#Generic Hash)
  And **NGINX Plus** adds two more methods:
  - [Least Response Time Method](#Least Response Time Method)

## Round Robin
Requests are distributed evenly across the servers, with server weights taken into consideration. This method is used by default (there is no directive for enabling it):
```
!(https://raw.githubusercontent.com/v2d27/nginx-config/main/Round-Robin%20Load%20Balancing)
```
## IP Hash
The server to which a request is sent is determined from the client IP address. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available.

## Least Connection
A request is sent to the server with the least number of active connections, again with server weights taken into consideration:

## Generic Hash
The server to which a request is sent is determined from a user‑defined key which can be a text string, variable, or a combination. For example, the key may be a paired source IP address and port, or a URI as in this example:

## Least Response Time Method
**Least Time (NGINX Plus only)** – For each request, NGINX Plus selects the server with the lowest average latency and the lowest number of active connections, where the lowest average latency is calculated based on which of the following parameters to the `least_time` directive is included:
`header` – Time to receive the first byte from the server
`last_byte` – Time to receive the full response from the server
`last_byte inflight` – Time to receive the full response from the server, taking into account incomplete requests


## Source
- [Using nginx as load balancer](https://nginx.org/en/docs/http/load_balancing.html)
- [Load Balancing Algorithms](https://www.geeksforgeeks.org/load-balancing-algorithms/)
- [HTTP Load Balancing NGINX](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
