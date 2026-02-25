# 1. Instalare Ubuntu Server (HWE kernel)

Sistemul a fost instalat cu Ubuntu Server LTS si HWE kernel.

Au fost realizate urmatoarele:

* configurare Wi-Fi in timpul instalarii
* finalizare instalare fara pachete Snap

Rezultat:

Server Linux functional pe laptop, cu conectivitate Wi-Fi stabila.

---

# 2. Configurare acces SSH

Ai instalat si activat serviciul `openssh-server`.

Conectarea se face din laptop cu comanda:

```bash
ssh theodor-pve@192.168.1.50
```

Rezultat:

Administrare remote completa a sistemului.

---

# 3. Configurare IP static prin Netplan

Ai configurat adresa statica prin fisierul `/etc/netplan/99-static.yaml`.

Setari aplicate:

```text
IP:        192.168.1.50
Gateway:   192.168.1.1
DNS:       192.168.1.1 + 8.8.8.8
```

Configuratia a fost aplicata cu `netplan apply`.

Rezultat:

* serverul foloseste permanent aceeasi adresa IP
* conexiunea SSH ramane stabila dupa reboot
* infrastructura este pregatita pentru servicii DNS si VM-uri

---

# 4. Instalare si configurare KVM

Componente instalate:

* `qemu-kvm`
* `libvirt-daemon-system`
* `virtinst`
* `virt-manager`

Utilizatorul a fost adaugat in grupurile:

* `kvm`
* `libvirt`

Rezultat:

* hypervisor functional
* retea NAT KVM activa (`virbr0`)
* mediu pregatit pentru creare de masini virtuale

---

# 5. Decizie de arhitectura pentru Wi-Fi si virtualizare

Alegerea de arhitectura este corecta pentru acest tip de hardware:

* Ubuntu Server + KVM, fara Proxmox bare-metal
* utilizare NAT pentru VM-uri in loc de bridge direct pe Wi-Fi

Rezultat:

Setup stabil si realist pentru un homelab.

---

# Rezumat executiv

Ai construit un server Linux minimal, cu IP static, acces SSH si virtualizare KVM, pregatit pentru servicii de infrastructura.

Servicii tinta:

* VM-uri
* DNS server
* Prometheus
* monitoring si automatizare

---

# Stare curenta

```text
Laptop
└─ Ubuntu Server (Wi-Fi, IP static 192.168.1.50)
        ├─ SSH
        ├─ KVM / libvirt
        └─ pregatit pentru DNS si monitoring
```

---

<!-- Creare VLAN in retea + VM infra -->
### Creare LAN: `lan-wifi`: un vor fi prezente toate VM-urile

Acest `lan-wifi` a fost creat pentru a putea face `bridge` intre masinile conectate pe o interfata `wifi` (server-ul bare metal este laptop conectat in retea prin WiFi, iar aceasta este singura metoda.

![config-lan](image.png)
![pornire-succes-lan](image-1.png)

# Creare VM de monitorizare si configurarea infrastructura cu cloud-init + cloud image:

Instalare utilitare:
```
sudo apt install -y cloud-image-utils libguestfs-tools wget
```

Unde:
- `cloud-image`: Este o imagine Ubuntu deja pregătită pentru VM: fără installer,minimală,pregătită să primească config la boot
- `cloud-init`: la PRIMUL boot al VM-ului:creează user, setează parola sau cheia SSH, setează hostname,instalează pachete,rulează comenzi
Aceasta imagine(ISO) este oficiala UBUNTU.

Download Ubuntu 22.04 ISO in folder-ul de infrastructura pentru a folosi la creearea VM-ului:

```
mkdir -p ~/vms/infra
cd ~/vms/infra
wget -O ubuntu.qcow2 https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
```

Se va descarca imaginea custom: `ubuntu.qcow2` -> imagine optimizata pentru a rula intr-un mediu virtualizat, oficiala Ubuntu, pre-configurata
Vom redenumi imaginea pentru a o putea refolosi + alocare de 50GB:
```
cp ubuntu.qcow2 infra.qcow2
qemu-img resize infra.qcow2 50G
```
Apoi vom configura VM-ul:

1. Generare cheie SSH pentru noul VM:
![gen-cheie-ssh](image-2.png)

2. Config `user-data` si `meta-data`:
![user-data](image-3.png)

Incarcare datele de configurare in imaginea ISO cloud-init `infra-seed.iso`: `cloud-localds infra-seed.iso user-data meta-data`
Aceasta imagine va face configurarea vazuta mai sus.

In crearea VM-ului vom incarca ambele ISO-uri:
- primul: imaginea cu OS `qcow2`
- al doilea: configul OS-ului mentionat: `infra-seed.iso`

Acestea sunt citite ca discuri -> asa functioneza citirea KVM-ului (hypervisor-ul de virtualizare a linux-ului)

### Creare VM: atasarea IOS-uri, configurare VM si introducerea in retea
In cazul virtualizatii cu KVM, avem prezente 2 retele:
1. Retea "Routerului" unde toate device-urile/VM-urile sunt conectate: `192.168.1.0/24`
2. O retea Virtuala NAT KVM

Pentru moment vom crea o singura interfata`(wlo1)`, conectata la reteaua router-ului (bridge realizat prin "reteaua" LAN `wifi-lan` creata anterior)

Comanda creare VM prin KVM:

```
sudo virt-install \
  --name infra \
  --memory 3072 \
  --vcpus 2 \
  --cpu host \
  --disk path=/var/lib/libvirt/images/infra/infra.qcow2,format=qcow2,bus=virtio \
  --disk path=/var/lib/libvirt/images/infra/infra-seed.iso,device=cdrom \
  --os-variant ubuntu22.04 \
  --network network=default,model=virtio \
  --graphics none \
  --import \
  --noautoconsole
```

And by tryng SSH:
![ssh-first-vm](image-5.png)

Arhitectura actuala:
```
Laptop admin (client SSH) - 192.168.1.146/24
        |
        |  LAN/Wi-Fi 192.168.1.0/24 (gateway ex. 192.168.1.1)
        |
Host KVM (Ubuntu Server) - 192.168.1.50/24
        |
        |  libvirt NAT network: 192.168.122.0/24 (Gateway pe if `virbr0`: 192.168.122.1)
        |
VM infra - 192.168.122.4/24 (NIC: enp1s0, DHCP)
```