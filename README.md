# codestar_kubernetes_meterials
# or create a new repository on the command line
```
echo "# codestar_kubernetes_meterials" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/rockman88v/codestar_kubernetes_meterials.git
git push -u origin main
```
# or push an existing repository from the command line
```
git remote add origin https://github.com/rockman88v/codestar_kubernetes_meterials.git
git branch -M main
git push -u origin main
```
# or import code from another repository
```
You can initialize this repository with code from a Subversion, Mercurial, or TFS project.
```

Cài đặt ArgoCD  trên K8S

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Cài đặt argocd cli trên control-plane
```
sudo curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo chmod +x /usr/local/bin/argocd
```

## Đăng nhập vào argocd từ cli
### Edit service cho argocd thành nodePort
```
ubuntu@base-node:~/lession12-gitops$ k -n argocd edit svc argocd-server
service/argocd-server edited
```
```
ubuntu@base-node:~/lession12-gitops$ k -n argocd get svc argocd-server
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
argocd-server   NodePort   10.110.139.93   <none>        80:31156/TCP,443:30094/TCP   13m
```
Lúc này có thể kết nối tới argocd thông qua IP của node và NodePort. Thông tin IP node có thể sử dụng private IP:
```
ubuntu@base-node:~/lession12-gitops$ k get node -owide
NAME               STATUS   ROLES           AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
ip-172-31-27-172   Ready    <none>          4h5m   v1.26.0   172.31.27.172   <none>        Ubuntu 20.04.5 LTS   5.15.0-1026-aws   containerd://1.6.14
ip-172-31-29-82    Ready    control-plane   4h6m   v1.26.0   172.31.29.82    <none>        Ubuntu 20.04.5 LTS   5.15.0-1026-aws   containerd://1.6.14
ip-172-31-29-96    Ready    <none>          4h5m   v1.26.0   172.31.29.96    <none>        Ubuntu 20.04.5 LTS   5.15.0-1026-aws   containerd://1.6.14
ip-172-31-31-1     Ready    <none>          4h5m   v1.26.0   172.31.31.1     <none>        Ubuntu 20.04.5 LTS   5.15.0-1026-aws   containerd://1.6.14
ip-172-31-31-184   Ready    <none>          4h5m   v1.26.0   172.31.31.184   <none>        Ubuntu 20.04.5 LTS   5.15.0-1026-aws   containerd://1.6.14
```
### Lấy thông tin đăng nhập argocd
Thực hiện lệnh:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Login argocd bằng cli
Thực hiện lệnh:
```
ubuntu@base-node:~/lession12-gitops$ argocd login 172.31.31.184:30094
WARNING: server certificate had error: x509: cannot validate certificate for 172.31.31.184 because it doesn't contain any IP SANs. Proceed insecurely (y/n)? y
Username: admin
Password:
'admin:login' logged in successfully
Context '172.31.31.184:30094' updated
```
Trong đó `172.31.31.184` là IP Private của một worker-node.

### Login argocd trên UI
Kết nối argoCD bằng Public IP của một workernode ở port NodePort

## Cấu hình repo cho argocd
### Tạo SSH key trên server control-plane (hoặc nơi cài argocd cli)
Tạo ssh-key bằng lệnh:
```
ubuntu@base-node:~/lession12-gitops$ ssh-keygen
Generating public/private rsa key pair.



Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:H6BLcJYLgo3sbI6T3yzRH4Uy+sbCk10+cN78NG5Q5Y8 ubuntu@base-node
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|.+     .    .    |
|o.o o +..  o     |
|o  .o=.o... .    |
| + o o+.S..  o   |
|+.o o.+... .E .  |
|+o * B.+ .+      |
| o*o= = oo..     |
|  .=o  . oo      |
+----[SHA256]-----+
```

### Option1: Sử dụng CodeCommit
**SSH key ID**
APKAVUY6XXFD2OENHAGZ

Cấu hình kết nối tới CodeCommit từ SSH ở client:
Tạo file `~/.ssh/config` có nội dung:
```
Host git-codecommit.*.amazonaws.com
  User APKAVUY6XXFD2OENHAGZ   #SSH key ID tạo trước đó
  IdentityFile /home/ubuntu/.ssh/id_rsa   #Link file private key
```
Gán đúng quyền cho file:
```
sudo chmod 600 ~/.ssh/config
```

Thực hiện clone code mẫu từ repo sau:
```
git clone https://github.com/kumcp/k8s_learning_example.git
```

Tạo một repo mới trên CodeCommit:
vd: demo

Clone repo này về máy client:
```
git clone  ssh://APKAVUY6XXFD2OENHAGZ@git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/demo
```

Copy nội dung thư mục code mẫu vào thư mục `demo` vừa pull về máy:
```
cp -r k8s_learning_example/* demo/
```
Kiểm tra trạng thái git:
```
ubuntu@base-node:~/lession12-gitops/demo$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        Doc/
        README.md
        awstest/
        cheatsheet.md
        example/

nothing added to commit but untracked files present (use "git add" to track)
```
Cấu hình email cho git:
```
git config --global user.email "rockman88v@gmail.com"
```
Sau đó add và commit:
```
git add .
git commit -m "add file to repo"
git push origin master
```

Sau đó thực hiện add repo này vào argocd server:
```
argocd repo add ssh://<SSH_KEY_ID>@git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/<code-commit-repo-name> --ssh-private-key-path /home/ubuntu/.ssh/id_rsa  --insecure-skip-server-verification
```

```
ubuntu@base-node:~/lession12-gitops/demo$ argocd repo add ssh://APKAVUY6XXFD2OENHAGZ@git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/demo --ssh-private-key-path /home/ubuntu/.ssh/id_rsa  --insecure-skip-server-verification

Repository 'ssh://APKAVUY6XXFD2OENHAGZ@git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/demo' added
```

Kiểm tra kết quả:
```
ubuntu@base-node:~/lession12-gitops/demo$ argocd repo list
TYPE  NAME  REPO                                                                                  INSECURE  OCI    LFS    CREDS  STATUS      MESSAGE  PROJECT
git         ssh://APKAVUY6XXFD2OENHAGZ@git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/demo  true      false  false  false  Successful
```


### Option2: Sử dụng github
