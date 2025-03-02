# airflow-materials-aws
Materials for the next course

## Links

- [4.16 Configuring the workstation](#Configuring-the-workstation)
- [4.17 Configuring Cloud9 with the Admin account](#Configuring-Cloud9-with-the-Admin-account)
- [4.20 Creating and configuring the Git repository for GitOps](#Creating-and-configuring-the-Git-repository-for-GitOps)
- [4.22 Creating the Cluster with eksctl](#Creating-the-Cluster-with-eksctl)
- [4.23 Installing and Configuring Flux](#Installing-and-Configuring-Flux)
- [5.30 Installing the EBS Driver](#Installing-the-EBS-Driver)

### Configuring the workstation

- Update packages of the instance

```bash
sudo yum -y update
```

- Create a python virtual environment

```bash
python3 -m venv .sandbox
```

- Active the python virtual environment

```bash
source .sandbox/bin/activate
```

- Upgrade pip

```bash
pip install --upgrade pip
```

- Download and extract the latest release of eksctl with the following command.

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

- Test that your installation was successful with the following command.

```bash
eksctl version
```

- Download the latest release of Kubectl with the command

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.24.8/bin/linux/amd64/kubectl
```

- Make the kubectl binary executable.

```bash
chmod +x ./kubectl
```

- Move the binary in to your PATH.

```bash
sudo mv ./kubectl /usr/local/bin/kubectl
```

- Test to ensure the version you installed is up-to-date:

```bash
kubectl version --client
```

- Install Helm3

```bash
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

- Check the version

```bash
helm version --short
```

- Download the stable repo

```bash
helm repo add stable https://charts.helm.sh/stable
```

- Config git (change the name by airflow-workstation and keep the email. Save and exit the file.)

```bash
git config --global user.name "airflow-workstation"
```

### Configuring Cloud9 with the Admin account

- upgrade aws cli

```bash
pip install --upgrade awscli && hash -r
```

- install some utilities

```bash
sudo yum -y install jq gettext bash-completion moreutils
```

- go the settings, AWS settings and turn off temporary credentials

- remove temporary credentials

```bash
rm -vf ${HOME}/.aws/credentials
```

- configure aws env variables
- The following get-caller-identity example displays information about the IAM identity used to authenticate the request

```bash
aws configure
```

```bash
aws sts get-caller-identity
```

```bash
export ACCOUNT_ID=
```

```bash
export AWS_REGION=
```

- update the file bash_profile and configure aws

```bash
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
```

```bash
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```

### Creating and configuring the Git repository for GitOps

- Press return for all questions by keeping the defaults and empty passphrase.

```bash
ssh-keygen -t rsa
```

- copy public key to github ssh keys

```bash
cat /home/ec2-user/.ssh/id_rsa.pub
```

### Creating the Cluster with eksctl

```bash
cd airflow-materials-aws
```

- Install the key

```bash
aws ec2 import-key-pair --key-name "airflow-workstation" --public-key-material fileb:///home/ec2-user/.ssh/id_rsa.pub
```

- Install the key (pennvet)

```bash
aws ec2 import-key-pair --key-name "airflow-workstation-pennvet" --public-key-material fileb:///home/ec2-user/.ssh/id_rsa.pub
```

- Install the key (johnvirtuon)

```bash
aws ec2 import-key-pair --key-name "airflow-workstation-john" --public-key-material fileb:///home/ec2-user/.ssh/id_rsa.pub
```

- Install aws-iam-authenticator
- Othwerise, doesn't look for the token and kubectl can't connect

```bash
curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.5.9/aws-iam-authenticator_0.5.9_linux_amd64
chmod +x ./aws-iam-authenticator
```

```bash
mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
aws-iam-authenticator help
```

- Install AWS V2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

```bash
unzip awscliv2.zip
sudo ./aws/install --update
```

```bash
aws --version
```

```bash
cd airflow-materials-aws
```

- Create the cluster

```bash
eksctl create cluster -f cluster.yml
```

- Check if the cluster is healthy

```bash
kubectl get nodes
```

```bash
kubectl get pods --all-namespaces
```

### Installing and Configuring Flux

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

```bash
flux --version
```

```bash
export GITHUB_TOKEN=<your_token_from_github>
```

```bash
echo $GITHUB_TOKEN
```

```bash
flux bootstrap github \
  --owner=paulkur \
  --repository=airflow-eks-config \
  --branch=main \
  --interval=15s \
  --personal
```

```bash
mkdir airflow-eks-config/{workloads,releases,namespaces}
find airflow-eks-config/ -type d -exec touch {}/.keep \;
```

```bash
cd airflow-eks-config
git add .
git commit -am "directory structure"
git push
```

- after adding namespaces

```bash
cd airflow-eks-config
git add .
git commit -am "add a new namespaces"
git push
```

################## Other account

- for personal aws johnvirtuon account

```bash
flux bootstrap github \
  --owner=paulkur \
  --repository=airflow-eks-config-john \
  --branch=main \
  --interval=15s \
  --personal
```

```bash
mkdir airflow-eks-config-john/{workloads,releases,namespaces}
find airflow-eks-config-john/ -type d -exec touch {}/.keep \;
```

```bash
cd airflow-eks-config-john
git add .
git commit -am "directory structure"
git push
```

- after adding namespaces

```bash
cd airflow-eks-config-john
git add .
git commit -am "add a new namespaces"
git push
```

- check namespaces

```bash
kubectl get namespaces
```

### Installing the EBS Driver

- oficial [git repo here](https://github.com/kubernetes-sigs/aws-ebs-csi-driver), [installation here](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md)
- commands to run

```bash
cd airflow-eks-config
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm repo update
```

```bash
helm upgrade --install aws-ebs-csi-driver \
    --namespace kube-system \
    aws-ebs-csi-driver/aws-ebs-csi-driver
```

## kubectl commands

- check pods

```bash
watch kubectl get pods -n dev
```

- enter pod

```bash
kubectl exec -it airflow-dev-scheduler-6bf9b46568-xrhgq -n dev -c scheduler -- /bin/bash
```

- list dags

```bash
airflow dags list
```

- delete all pods

```bash
kubectl delete pods --all --force --grace-period=0 -n dev
```

### Flux commands

```bash
flux get all
```

- filter error messages

```bash
flux logs --follow --level=error --all-namespaces
```
