# Helm Playground

Bu repository, **Helm kullanarak Kubernetes (k3s / Minikube)** üzerinde uygulama keşfi, yapılandırması, versiyonlama yönetimi ve tam kapsamlı bir WordPress kurulumu çalışmalarını içermektedir.

---

## 1. Vagrant Ortamının Başlatılması

İlk olarak sanal makineler ayağa kaldırılır:

```bash
vagrant up 
```

Ardından K3s server node’una bağlanılır:

```bash
vagrant ssh server-0
```

---

## 2. Kubeconfig Dosyasının Ayarlanması (Remote Access)

Cluster’ı kendi bilgisayarımızdan yönetebilmek için kubeconfig dosyası yapılandırılır.

Server node üzerinde:

```bash
cat /etc/rancher/k3s/k3s.yaml
```

Komut çıktısının **tamamı kopyalanır**.

Server’dan çıkılır:

```bash
exit
```

Kendi bilgisayarımızda kubeconfig dosyası açılır:

```bash
vim ~/.kube/config
```

Kopyalanan içerik buraya yapıştırılır.

### Server IP Güncellemesi

Aşağıdaki satırda bulunan `server` adresi:

```yaml
server: https://127.0.0.1:6443
```

Vagrantfile’da tanımlanan **server node’un private static IP adresi** ile değiştirilir.  
Bu işlem, VM dışından cluster’a erişim sağlamak için gereklidir.

---

## 3. Helm Chart Seçimi

Helm chart’ları incelemek için:

- https://artifacthub.io/

Bu çalışma kapsamında **Bitnami NGINX chart’ı** tercih edilmiştir.

---

## 4. Bitnami Helm Repository Ekleme

Bitnami repository ekli değilse aşağıdaki komutlar çalıştırılır:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

## 5. NGINX Helm Chart Deploy Edilmesi

NGINX, belirli bir versiyon ile deploy edilir:

```bash
helm install my-nginx bitnami/nginx --version 22.4.3
```

Özel değerler (values) kullanılmak istenirse:

```bash
helm install my-nginx bitnami/nginx \
--version 22.4.3 \
-f /localpath/values/nginx_value.yaml
```

---

## 6. Helm NOTES Bölümü Nedir?

Helm install işleminden sonra görünen **NOTES** bölümü;

- Uygulamaya erişim yöntemleri  
- Port ve servis bilgileri  
- DNS yönlendirmeleri  
- Chart’a özel uyarılar  

gibi bilgileri içerir.

Bu alan, **chart geliştiricisi tarafından** kullanıcıyı yönlendirmek amacıyla tanımlanır.

---

## 7. Helm Upgrade ile Kaynak Güncellemesi

İstenen görev:

> Uygulamanın 1 CPU ve 2 GB memory ile 3 replica çalışacak şekilde  `helm upgrade` ile güncellenmesi

Sistem kaynaklarım yeterli olmadığı için uygulama aşağıdaki değerlerle güncellenmiştir:

- **1 CPU**
- **512Mi Memory**

### Örnek `values.yaml`

```yaml
resources:
  requests:
    cpu: "1"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
replicaCount: 2
```

### Upgrade İşlemi

```bash
helm upgrade my-nginx bitnami/nginx \
--version 22.4.3 \
-f /localpath/values/nginx_value.yaml
```

---


