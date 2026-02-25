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

