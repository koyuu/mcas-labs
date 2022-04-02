docker exec -it <collector name> bash
docker ps
docker inspect 

 docker login -u caslogcollector -p C0llector3nthusiast

 (echo 6f19225ea69cf5f178139551986d3d797c92a5a43bef46469fcc997aec2ccc6f) | docker run --name MyLogCollector -p 21:21 (port mapping) -p 20000-20099:20000-20099 -e "PUBLICIP='192.2.2.2'" -e "PROXY=192.168.10.1:8080" -e "CONSOLE=tenant2.eu1-rs.adallom.com" -e "COLLECTOR=MyLogCollector" --security-opt apparmor:unconfined --cap-add=SYS_ADMIN --restart unless-stopped -a stdin -i mcr.microsoft.com/mcas/logcollector starter


 ftpd '-p <first port>:<last port>': Use only ports in the range <first port>
to <last port> inclusive for passive-mode downloads. This is especially
useful if the server is behind a firewall without FTP connection tracking.
Use high ports (40000-50000 for instance), where no regular server should be
listening.

command=/usr/local/sbin/pure-ftpd -iAjHER --tls=1 --tlsciphersuite=HIGH -k 90 -P '192.2.2.2' -p 20000:20099 -l puredb:/etc/pureftpd.pdb -O clf:/var/log/pure-ftpd/transfer.log

-A': chroot() everyone, but root. There's no such thing as a trusted
group. '-A' and '-a <gid>' are mutually exclusive.
'-E': Only allow authenticated users. Anonymous logins are prohibited.
-i': Disallow upload for anonymous users, whatever directory permissions
are. This option is especially useful for virtual hosting, to avoid your
users creating warez sites in their account.

'-j': If the home directory of a user doesn't exist, automatically create
it. The newly created home directory belongs to the user and permissions are
set according to the current directory mask. Only the home directory can be
created (so /home/john/./public_html won't work, but /home/john will) . To
avoid local attacks, the parent directory should never belong to an untrusted
user. Also note that you must trust whoever manages the users databases,
because with that feature, he'll be able to create/chown directories anywhere
on the server's filesystem.


When the previous command is run, the server will listen for incoming
connections on every interface, all IP addresses and the standard FTP port
(21) . If your system has IPv6 addresses, they should work as well.

Now, if you want to listen for an incoming connection on a non-standard port,
just append '-S' and the port number:

/usr/local/sbin/pure-ftpd -S 42

Service names are also allowed ('-S smtp' and the daemon will be accepting
connections on the SMTP port (25) . Very uncommon, but we should please
everybody anyway, even disturbed minds) .

Now, what if your system has many IP addresses and you want the FTP server
to be reachable on only one of these addresses, let's say 192.168.0.42?
Just use the following command line:

/usr/local/sbin/pure-ftpd -S 192.168.0.42,

The final comma is important, don't forget it. Actually, it's a shorthand for:

/usr/local/sbin/pure-ftpd -S 192.168.0.42,21

If you prefer host names over IP addresses, it's your choice:

/usr/local/sbin/pure-ftpd -S ftp.example.com,21



Have you ever been in a situation where you forgot to “expose” a port for your container, or you’d like to change the port mapping for an existing container? I know I have been!!

When you perform a quick google search, most common answer’s are

you cannot do that! You’ll have to create a new container with proper port mapping
create an image of your existing container. Then use “docker run” with your desired port mapping for your “newly created” image.
Among those answers, a saint would have mentioned – it’s possible. But you’d have to do some extra work. So here I’m, telling you – yep, it’s possible. I have tried it and it works. For steps, see the linked answer written by “holdfenytolvaj”.

Here, I’ll explain, what needs to be changed in order for you to modify port mapping. I would like to (in my case) expose port 8888 from my docker container.

In my case, I would like to expose an additional port – 8888 – from my docker container.

Step 1: Using “docker inspect” get details about current port mapping. This will be seen under “NetworkSettings”. And “PortBindings” under “HostConfig”.

"Ports": {
 "80/tcp": [ 
{
 "HostIp": "0.0.0.0",
 "HostPort": "80"
 }
]
 },
 

The above snippet (from NetworkSettings.Port) declares – expose port 80 from my docker container to port 80 (on every network device) in my docker host machine.

NOTE: Stop the container and docker engine before editing the below files.

Step 2:  Edit the config.v2.json file as shown below

(a) Update entry for “ExposedPorts”

(b) Update entry for “Ports”

$ vi /var/lib/docker/containers//config.v2.json
...
{
"Config": {
....
"ExposedPorts": {
"80/tcp": {},
"8888/tcp": {}
},
....
},
"NetworkSettings": {
....
"Ports": {
 "80/tcp": [
 {
 "HostIp": "",
 "HostPort": "80"
 }
 ],
 "8888/tcp": [
 {
 "HostIp": "",
 "HostPort": "8888"
 }
 ]
 },
....
}
In the above snippet, I have included one more port – 8888 –  to be exposed as *:8888 on my host machine.

Step 3:  Edit the hostconfig.json file as shown below

(a) Update entry for “PortBindings”

$ vi /var/lib/docker/containers//hostconfig.json
{
....
 "PortBindings": {
 "80/tcp": [
 {
 "HostIp": "",
 "HostPort": "80"
 }
 ],
 "8888/tcp": [
 {
 "HostIp": "",
 "HostPort": "8888"
 }
 ]
 },
.....
}
Advertisements

REPORT THIS AD

Save the file. Re-start your docker engine (docker service via systemctl). Verify docker engine has started successfully, without any errors.

Start your container.

When you execute “docker ps” command, the PORTS column should show the updated port mapping details.