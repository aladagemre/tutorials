# LetsEncrypt HAProxy with DNS challenge 

In this post, I will be showing you how to issue LetsEncrypt SSL certificate for your HAProxy running on Docker container. We will be using DNS challenge to verify we own the domains. hook.rb script will connect to Amazon Route 53 to update DNS records for this reason.

letsencrypt.sh will start the challenge and start hook.rb to update DNS records and then check if records are updated.

```bash
git clone https://github.com/lukas2511/letsencrypt.sh
cd letsencrypt.sh
cp domains.txt.example domains.txt
cp config.sh.example config.sh

wget https://gist.githubusercontent.com/tache/3b6760784c098c9139c6/raw/33fe6e0791a7d40ce7cdf14019b7d31801d4ab05/hook.rb
sudo apt-get install ruby
sudo gem install aws-sdk pry awesome_print domainatrix
```

Then set your environment variables that can modify Amazon Route 53:

```bash
export AWS_ACCESS_KEY_ID="xx"
export AWS_SECRET_ACCESS_KEY="xx"
```
Edit config.sh and set the following:

```bash
CHALLENGETYPE="dns-01"
HOOK="/home/ubuntu/letsencrypt.sh/hook.rb"
CONTACT_EMAIL="your@email.com"
```
Edit domains.txt:

    main.domain.com sub1.main.domain.com sub2.main.domain.com sub3.main.domain.com

Then run the script to issue the certificate.

```bash
./letsencrypt.sh -c -k /home/ubuntu/letsencrypt.sh/hook.rb
cd /home/ubuntu/letsencrypt.sh/certs/main.domain.com/
cat privkey.pem fullchain.pem > certificate.pem
sudo cp /etc/haproxy/certificate.pem /etc/haproxy/certificate.pem.old
sudo cp certificate.pem /etc/haproxy/certificate.pem
```

Now make sure you have certificate.pem and SSL configured haproxy.cfg in /etc/haproxy.cfg. You can add SSL support with:

```
frontend marathon_https_in
  bind *:443 ssl crt /etc/haproxy/certificate.pem
```

Then start your HAProxy container with Docker image:
```bash
docker run -d -p 443:443 -p 80:80 --name haproxy -v /etc/haproxy:/etc/haproxy:ro -v /dev/log:/dev/log million12/haproxy
#docker restart haproxy
```

