# allora
1. Install
sudo apt-get update
sudo apt-get install -y make gcc jq

sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add –
sudo add-apt-repository “deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable”
apt-cache policy docker-ce
sudo apt install docker-ce -y
