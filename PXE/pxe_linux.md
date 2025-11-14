# Guia Completo: Criar Imagem do Windows + Deploy via PXE Usando Servidor Linux

Este guia explica como criar uma imagem do Windows personalizada com softwares pré-instalados, generalizá-la com **Sysprep**, capturá-la e disponibilizá-la via PXE utilizando um **servidor Linux** (Ubuntu/Debian/CentOS/RHEL).

O fluxo aqui usa:

* **dnsmasq** (DHCP + TFTP)
* **iPXE** (carregador de rede)
* **wimboot** (para bootar Windows PE em modo UEFI)

---

# 🧱 1. Preparar a Máquina Windows de Referência

1. Instale o Windows normalmente.
2. Instale programas, drivers e configurações.
3. NÃO ingresse no domínio.
4. Aplique todas as configurações padrão.

---

# 🔧 2. Rodar o Sysprep para Generalizar

Abra o Sysprep:

```
C:\Windows\System32\Sysprep\sysprep.exe
```

Configurações:

* **OOBE** ✓
* **Generalize** ✓
* **Shutdown**

Ou via linha de comando:

```
sysprep /generalize /oobe /shutdown
```

Após desligar, a imagem está pronta para ser capturada.

---

# 📦 3. Capturar a Imagem Windows (install.wim) com WinPE + DISM

Inicialize a máquina com WinPE (pendrive, iPXE ou ISO).

Liste volumes:

```
diskpart
list vol
exit
```

Capture a imagem:

```
Dism /Capture-Image \
  /ImageFile:D:\CustomWindows.wim \
  /CaptureDir:C:\ \
  /Name:"Windows Custom"
```

Copie o arquivo **CustomWindows.wim** para o servidor Linux.

---

# 🖥️ 4. Estrutura de Pastas no Servidor Linux

Crie o diretório para armazenar imagens:

```
sudo mkdir -p /srv/pxe/{boot,iso,images}
```

Coloque:

* `boot/` → arquivos do iPXE
* `iso/` → arquivos do WinPE (boot.wim)
* `images/` → seus *.wim*

Exemplo:

```
/srv/pxe/
 ├── boot/
 │    └── wimboot
 ├── iso/
 │    └── boot.wim
 └── images/
      └── CustomWindows.wim
```

---

# 🌐 5. Instalar e Configurar dnsmasq (DHCP + TFTP + PXE)

Instale:

```
sudo apt install dnsmasq -y
```

Edite o arquivo:

```
sudo nano /etc/dnsmasq.d/pxe.conf
```

Conteúdo recomendado para UEFI:

```
port=0

# DHCP
interface=enp0s3
bind-interfaces

# Range DHCP
dhcp-range=192.168.1.50,192.168.1.150,12h

# Boot UEFI
dhcp-match=set:efi64,option:client-arch,7

dhcp-boot=tag:efi64,ipxe.efi

# TFTP
enable-tftp
tftp-root=/srv/pxe
```

Reinicie:

```
sudo systemctl restart dnsmasq
```

---

# 🚀 6. Preparar o iPXE

Baixe o **wimboot**:

```
wget https://github.com/ipxe/wimboot/releases/latest/download/wimboot -O /srv/pxe/boot/wimboot
```

Crie o script principal do iPXE:

```
sudo nano /srv/pxe/boot/boot.ipxe
```

Conteúdo:

```
#!ipxe

kernel boot/wimboot
initrd iso/boot.wim    boot.wim
initrd images/CustomWindows.wim custom.wim
boot
```

Coloque o binário ipxe.efi:

```
wget https://boot.ipxe.org/ipxe.efi -O /srv/pxe/boot/ipxe.efi
```

---

# 🧪 7. Testar o Boot PXE UEFI

1. Configure a BIOS em **UEFI Only**
2. Ative **Network Boot (IPv4 UEFI)**
3. A máquina deve carregar:

```
iPXE initialising devices...
```

E depois carregar o WinPE seguido da sua imagem customizada.

---

# 🐍 8. Automação com Python (Enviar Imagem p/ Servidor Linux)

Exemplo simples:

```python
import paramiko
from scp import SCPClient

local_file = "C:/capturas/CustomWindows.wim"
remote_file = "/srv/pxe/images/CustomWindows.wim"

ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect("192.168.1.10", username="root", password="SENHA")

with SCPClient(ssh.get_transport()) as scp:
    scp.put(local_file, remote_file)

print("Imagem enviada com sucesso!")
```

---

# 🏁 Conclusão

Com esse processo você terá:

* Uma imagem Windows personalizada
* Sysprep aplicado corretamente
* Servidor Linux funcionando como PXE completo
* Boot UEFI via iPXE + Wimboot
* Deploy totalmente funcional
