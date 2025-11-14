# Guia Completo: Criar Imagem Padrão do Windows + Deploy via PXE (UEFI)

Este guia explica, passo a passo, como preparar uma imagem do Windows com softwares pré-instalados, generalizar com *Sysprep*, capturar a imagem e disponibilizar para instalação via PXE em ambiente UEFI usando Windows Server.

---

## 📌 **1. Instalar o Windows e Configurar o Ambiente Base**

1. Instale o Windows normalmente em uma VM ou máquina de referência.
2. Instale **todos os softwares**, drivers e configurações desejadas.
3. Não conecte a máquina ao domínio.
4. Não personalize configurações de usuário (serão apagadas).

---

## 📌 **2. Limpar a Máquina Antes do Sysprep**

Execute estas ações para evitar erros no Sysprep:

* Desinstale antivírus provisórios.
* Remova perfis que não serão usados.
* Execute:

```
cleanmgr
```

* Desinstale Windows Store Apps desnecessários.

---

## 📌 **3. Executar o Sysprep (Generalizar)**

Abra o CMD como administrador e execute:

```
C:\Windows\System32\Sysprep\sysprep.exe
```

Configurações:

* **System Cleanup Action:** *Enter System Out-Of-Box Experience (OOBE)*
* **Generalize:** ✔️ marcado
* **Shutdown:** *Shutdown*

Depois clique em **OK**.

A máquina será desligada já generalizada.

---

## 📌 **4. Capturar a Imagem (Install.wim) pelo Windows ADK/WinPE**

1. Inicialize a máquina com um pendrive WinPE ou via rede.
2. No prompt do WinPE, identifique a partição Windows:

```
diskpart
list vol
exit
```

3. Capture a imagem:

```
Dism /Capture-Image /ImageFile:D:\CustomWindows.wim /CaptureDir:C:\ /Name:"Windows Base"
```

A imagem será salva como **CustomWindows.wim**.

---

## 📌 **5. Copiar a Imagem para o Servidor PXE**

No Windows Server:

* A pasta destino será:

```
/windows/iso/pxe/
```

Copie o arquivo **CustomWindows.wim** para a pasta correta:

```
/windows/iso/pxe/images/CustomWindows.wim
```

---

## 📌 **6. Configurar o WDS ou MDT para Deploy PXE (UEFI)**

### **No Windows Server:**

1. Instale o **Windows Deployment Services**.
2. Ative o modo **Deployment Server**.
3. Configure o WDS para responder requisições PXE.
4. Adicione a imagem de boot:

   * Boot.wim (do ISO do Windows)
5. Adicione a imagem de instalação:

   * CustomWindows.wim

### **Configurações UEFI:**

No WDS, marque:

* ✔️ *Enable UEFI (x64)*

---

## 📌 **7. Configurar o DHCP para PXE UEFI**

Adicione as opções no DHCP:

* **Option 66:** IP do WDS
* **Option 67 (UEFI):**

```
boot\x64\wdsmgfw.efi
```

---

## 📌 **8. Testar o Boot PXE**

1. Configure a BIOS da máquina para UEFI.
2. Ative *Network Boot / IPv4 UEFI*.
3. A máquina exibirá:

```
Press F12 for network boot
```

4. Após conectar no WDS, selecione sua imagem **CustomWindows.wim**.

O Windows será instalado já com todos os softwares configurados.

---

## 📌 **9. (Opcional) Automação com Python**

Exemplo simples para copiar a imagem automaticamente para a pasta PXE:

```python
import shutil
import os

origem = r"C:\capturas\CustomWindows.wim"
destino = r"\\servidor\\windows\\iso\\pxe\\images\\CustomWindows.wim"

os.makedirs(os.path.dirname(destino), exist_ok=True)
shutil.copy2(origem, destino)
print("Imagem copiada com sucesso!")
```

---

## ✔️ Conclusão

Com isso você terá:

* Uma imagem Windows padrão
* Softwares pré-instalados
* Sysprep aplicado
* Imagem implantável via PXE/UEFI
* Infraestrutura Windows Server funcionando como servidor de deploy

---
