````md
# 🚀 Install K3s + Rancher di Ubuntu (Step-by-Step)

Panduan ini membantu kamu setup:
- Kubernetes ringan (K3s)
- Rancher UI
- SSL via cert-manager
- Akses domain gratis (nip.io)

---

## 🧱 0) Prasyarat

Pastikan environment kamu:

- Ubuntu 20.04 / 22.04
- RAM ≥ 4 GB (disarankan ≥ 8 GB)
- AWS Security Group membuka port:
  - `22` (SSH)
  - `80` (HTTP)
  - `443` (HTTPS)
  - `6443` (Kubernetes API)
  - `2379-2380` (etcd)
  - `10250` (kubelet)
  - `8472` (Calico VXLAN)

---

## 🚀 1) Install Kubernetes (K3s)

```bash
curl -sfL https://get.k3s.io | sh -
````

Cek node:

```bash
sudo k3s kubectl get nodes
```

✔️ Status harus **Ready**

---

## ⚙️ 2) Setup kubectl tanpa sudo

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=$HOME/.kube/config
```

(Optional) Hapus env lama:

```bash
unset KUBECONFIG
```

Test:

```bash
kubectl get nodes
```

---

## 📦 3) Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Cek:

```bash
helm version
```

---

## 🔐 4) Install cert-manager (WAJIB)

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.crds.yaml
```

Tambahkan repo Helm:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

Install cert-manager:

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace
```

Cek:

```bash
kubectl get pods -n cert-manager
```

✔️ Semua pod harus **Running**

---

## 🌐 5) Setup Domain Gratis (nip.io)

Gunakan format:

```
<IP_ADDRESS>.nip.io
```

Contoh:

```
47.129.200.182.nip.io
```

✔️ Tidak perlu beli domain
✔️ Otomatis resolve ke IP

---

## 📦 6) Install Rancher via Helm

Tambahkan repo:

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
```

Buat namespace:

```bash
kubectl create namespace cattle-system
```

Install Rancher:

```bash
helm upgrade --install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=47.129.200.182.nip.io \
  --set bootstrapPassword=admin123
```

---

## ⏳ 7) Tunggu Rancher Ready

```bash
kubectl -n cattle-system get pods
```

✔️ Tunggu hingga:

```
rancher-xxxxx   Running
```

---

## 🌐 8) Akses Rancher UI

Buka di browser:

```
https://47.129.200.182.nip.io
```

---

## 🔐 9) Ambil Password Login

```bash
kubectl get secret --namespace cattle-system bootstrap-secret \
  -o go-template='{{.data.bootstrapPassword|base64decode}}{{"\n"}}'
```

Login:

* Username: `admin`
* Password: hasil command di atas

---

## ⚙️ 10) Setup Awal Rancher

* Ganti password
* Set Server URL:

```
https://13.212.216.8.nip.io
```

* Centang agreement
* Klik **Continue**

---

## 🎉 Selesai!

Sekarang kamu sudah punya:

* ✅ Kubernetes (K3s)
* ✅ Rancher UI
* ✅ cert-manager (SSL ready)
* ✅ Domain gratis (nip.io)
* ✅ Akses via browser

---

## 📌 Tips Tambahan

* Gunakan VM terpisah untuk production:

  * Control Plane
  * etcd
  * Worker
* Gunakan Load Balancer untuk high availability
* Backup etcd secara berkala

---

## 📚 Referensi

* [https://k3s.io](https://k3s.io)
* [https://rancher.com](https://rancher.com)
* [https://helm.sh](https://helm.sh)
* [https://cert-manager.io](https://cert-manager.io)

```

Kalau nanti kamu mau, saya bisa bantu bikin:
- versi **HA (3 control plane + external etcd)**
- atau tambahin **diagram arsitektur (biar repo kamu keliatan profesional)**
```
