# DNS as a Service in Virtual Private Cloud

This Cloud DNS is a reliable,scalable and managed hierarchial DNS service that can be used in a Virtual Private Cloud Environment to set up Private Hosted Zones ie)DNS servers at various points in the private L2 network. The created DNS system enables the usage of private domain names for the servers/machines inside the VPC and translates the requests for the domain names into their corresponding IP addresses. 

The DNS solution is an automated application that enables a VPC owner to configure DNS servers as per requirement in various subnets. The application auto-generates Forward and Reverse zone files for the subnets in the DNS server and enables access of servers in the VPC through URLs in addition to their IP addresses. This application can also configure DNS service across multiple sites that form a part of the VPC through GRE Tunneling. 
System Architecture: 
The system architecture contains two separate hypervisors with L2 Bridges to which Virtual Machines and namespaces are connected to. The Virtual Machines and Namespaces act as the clients , servers and DNS servers. GRE Tunneling is used to connect the two hypervisors to make it appear as if they exist in the same Private Network. 

#### Feature Architecture: 

1. Name Resolution within Zones
DNS server resolves the name and provides corresponding IP address configured. For example, Client 1 contacts the DNS to reach the ece server using www.ece.univ.edu. DNS resolves the address by giving the IP of any of the configured ece server. Now the client can directly reach the ece server using the obtained IP address.

2. Load Balancing of Servers
This feature balances traffic between several web servers that carry the same content/run the same application. The DNS server to which the request is made does load balancing and distributes the request by returning an IP address  from the set mapped to the same domain name in a round robin basis.

3. Location/Source IP Based Aware Resolution	
When the VPC offers a service / runs an application on a number of identically configured servers that are distributed across several subnets for redundancy / easy access purpose and the servers need to be reachable using a single domain name , but at the same time the DNS request needs to point users to the server that is closer / present in the same subnet as the client itself we use a Location Aware / Content Based Resolution.

4. High Availability / DNS Failover
When the client queries a DNS and finds that it is shutdown/ failed, then the next entry of DNS in /etc/resolv.conf file is queried for the name resolution.

#### Implementation: 

Clients and Web Servers :
Clients and Web Servers are either VMs installed using libvirt module on the L2 bridges or namespaces created in the hypervisor and attached to the L2 bridge using veth pairs

DNS Server:
BIND name server software is used to set up an internal DNS server which can resolve private host names to their IP addresses. Clients within the same or different subnet contact this DNS server to reach the web servers.

Router:
The Router is implemented as a  virtual machine with static routes that enable routing between different subnets in the VPC. 

#### Feature Implementation:

1. Resolution within Zones:
The DNS server that resides in a subnet is configured such that it is capable of resolving the IP addresses of Web Servers that reside in other subnets within the Private Network.
The steps involved are:
      * The different zones/subnets are listed in the named.conf file.
      * Separate forward and reverse resolution files for each subnet are created with mapping entries of the servers present in that subnet.
      * Replicate the forward and reverse resolution files in all the DNS servers that needs to do the DNS resolution
      * Configure the client’s resolv.conf file to point to the name server present in the same subnet / different subnet.

2. Load Balancing of Servers:
    *  Add the IP addresses of the servers under the same domain name as records in the Forward and reverse resolution zone files.
    * Replicate the forward and reverse resolution files in all the DNS servers that needs to do the DNS resolution
    * Configure the client’s resolv.conf file to point to the required DNS server.

3. Location / Source IP Based Resolution :
Implemented the Location Aware resolution with the help of views. Views allow to define different virtual configurations within the same server and also to specify which client should be sent which IP addresses as response for a DNS resolution request.

4. High Availability / DNS Failover
    * A secondary DNS server is configured with same configurations in the named.conf file and the database files for each zone.
    * The client’s resolv.conf should contain both the name server’s IP address as records to look for whenever a resolution request is sent out.

5. Phishing Prevention:
    * A namespace that acts a client whose DNS resolution requests should be dropped is created.
    * The ip tables of the DNS server is modified by adding an INPUT chain rule.
	Iptables -I INPUT -p udp --dport 53 -s <client ip> -j DROP
	Iptables -I INPUT -p tcp  --dport 53 -s <client ip> -j DROP
    * Configure the client’s resolv.conf file to point to the DNS server
    * When the client makes a request, the DNS server now drops the request.

#### Containers : 
DNS servers are  containers with a custom Centos image that has all tools installed including bind-utils, systemctl etc

#### Automation:
To automate the entire DNS configuration inside the VPC, a Python application is implemented that gets input from the user through a Northbound interface and communicates the data to the DNS configuration Ansible through a south bound interface. 

