# Create Client VPN

## Create certificates

``` shell
sudo yum update -y
sudo yum install -y git

git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa/easyrsa3

export EASYRSA_BATCH=1

./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa build-server-full server nopass
./easyrsa build-client-full client1.domain.tld nopass

mkdir ~/vpn-certs/
cp pki/ca.crt ~/vpn-certs/
cp pki/issued/server.crt ~/vpn-certs/
cp pki/private/server.key ~/vpn-certs/
cp pki/issued/client1.domain.tld.crt ~/vpn-certs
cp pki/private/client1.domain.tld.key ~/vpn-certs/
cd ~/vpn-certs/

aws acm import-certificate --certificate fileb://server.crt --private-key fileb://server.key --certificate-chain fileb://ca.crt
aws acm import-certificate --certificate fileb://client1.domain.tld.crt --private-key fileb://client1.domain.tld.key --certificate-chain fileb://ca.crt
```

[AWS Documentation](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/client-authentication.html#mutual)

## Create Client VPN in management console

## Create OpenVPN configuration file

``` shell hl_lines="3"
cd ~

aws ec2 export-client-vpn-client-configuration --client-vpn-endpoint-id <client vpn endpoint id> --output text > client-config.ovpn

sed -i '34 i<cert>\n</cert>\n<key>\n</key>' client-config.ovpn
sed -i "/<cert>/r /home/ec2-user/vpn-certs/client1.domain.tld.crt" client-config.ovpn
sed -i "/<key>/r /home/ec2-user/vpn-certs/client1.domain.tld.key" client-config.ovpn
```

[AWS Documentation](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/cvpn-working-endpoint-export.html)