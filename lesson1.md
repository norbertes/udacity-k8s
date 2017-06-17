# Configure Your Cloud Shell Environment
```sh
gcloud compute zones list
gcloud config set compute/zone europe-west1-d
```
# Download Go:
```sh
wget https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.6.2.linux-amd64.tar.gz
echo "export GOPATH=~/go" >> ~/.bashrc
source ~/.bashrc
```

# Get the code:
```sh
mkdir -p $GOPATH/src/github.com/udacity
cd $GOPATH/src/github.com/udacity
git clone https://github.com/udacity/ud615
```

# MSA hello world
```sh
# 1st shell
go build -o ./bin/hello ./hello
sudo ./bin/hello -http 0.0.0.0:10082

# 2nd shell
go build -o ./bin/auth ./auth
sudo ./bin/auth -http :10090 -health :10091

# test
TOKEN=$(curl 127.0.0.1:10090/login -u user | jq -r '.token')
curl -H "Authorization:  Bearer $TOKEN" http://127.0.0.1:10082/secure
```
