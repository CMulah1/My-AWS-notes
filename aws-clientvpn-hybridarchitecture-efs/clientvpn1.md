# aws-clientvpn-hybridarchitecture-efs/clientvpn_commands.txt

This is not a complete list of commands.  These commands are provided so that you can copy and paste the commands during the lab activity.

aws configure: set region to us-east-1
sudo yum update -y
sudo yum install -y git
git clone https://github.com/OpenVPN/easy-rsa.git  (Copying the files that we need to generate our certificates)
cd easy-rsa/easyrsa3  (change into that directory and run the command to generate our CA certificates)
./easyrsa init-pki (as we create our certificates, we need to create down with "nopassword" )
./easyrsa build-ca nopass (once done we will go ahead and build our server certificate and key)
./easyrsa build-server-full server nopass
./easyrsa build-client-full client1.domain.tld nopass


Make a directory for the certificates
mkdir /home/cloud_user/certificates


Create the CA, server and client certificates 
cp pki/ca.crt /home/cloud_user/certificates
cp pki/issued/server.crt /home/cloud_user/certificates
cp pki/private/server.key /home/cloud_user/certificates
cp pki/issued/client1.domain.tld.crt /home/cloud_user/certificates
cp pki/private/client1.domain.tld.key /home/cloud_user/certificates


Change to the certificates directory and import certificates into certificate manager in AWS.
cd /home/cloud_user/certificates


Import the certificates
sudo aws acm import-certificate --certificate file://server.crt --private-key file://server.key --certificate-chain file://ca.crt --region us-east-1 (Server key generated - make sure you take notes of the last 4 characters of the key which will be used to distinguished between both server key and client key when we go into creating the VPN endpoint).

sudo aws acm import-certificate --certificate file://client1.domain.tld.crt --private-key file://client1.domain.tld.key --certificate-chain file://ca.crt --region us-east-1 (Certficates are now created).


Export the client configuration (replace the endpoint id with the endpoint id from the Client VPN console)
aws ec2 export-client-vpn-client-configuration --client-vpn-endpoint-id endpoint_id --output text>client-config.ovpn 
(hit enter, and we've now downloaded the configuration file from the VPN endpoint).

Append the client certificate and key to the open vpn config file
cat >> client-config.ovpn 
cert /path/client1.domain.tld.crt 
key /path/client1.domain.tld.key 


Install OpenVPN client
sudo yum install -y openvpn

Launch openvpn client using the config file
sudo openvpn --config client-config.ovpn (typo in the file)


Make the mount directory
sudo mkdir /mnt/efs

Mount EFS
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport mount-target-IP:/   /mnt/efs

