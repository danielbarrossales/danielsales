---
title: "Instalando Arch Linux com Btrfs"

date: 2021-04-08T11:46:29-03:00
url: instalacao-arch-linux-com-btrfs/
image: images/2021-thumbs/archlinux-installation-with-btrfs.png
categories:
  - Linux
tags:
  - Archlinux
  - Btrfs
draft: false
---
## Instalação do sistema
Neste tutorial assumo que você já tenha uma mídia de instalação pronta e sabe como inicializar o sistema por ela.
<!--more-->
### 1. Configuração do ambiente
#### Mapeamento de teclas
O mapa de teclas padrão do console é US, mas ele pode ser facilmente alterado de com o comando `loadkeys`. Para alterar o mapa de teclas para o padrão brasileiro podemos utilizar o seguinte comando:
{{< highlight bash >}}
loadkeys br-abnt2
{{</ highlight >}}

Para saber quais mapas de teclas estão disponíveis no seu sistema:
{{< highlight bash >}}
localectl list-keymaps
{{</ highlight >}}

### 2. Partições do disco
#### Listar discos
O comando `lsblk` retornará os discos e partições com dados extras, como tamanho e ponto de montagem. 
#### Criar partições
Nesse ponto temos duas opções, que eu conheço, de comandos que podem ser utilizados: `fdisk` e `cfdisk`. Dentre essas duas opções `cfdisk` é mais amigável a novatos, já que é uma aplicação com interface gráfica.
Uma vez que tenha escolhido o disco que irá utilizar para instalar o archlinux é só utilizar um dos comandos acima seguido pelo nome do disco. Ex.:
{{< highlight bash >}}
cfdisk /dev/sda
{{</ highlight >}}
O objetivo é termos duas partições, primeiro a partição de boot com 512MB, a segunda com o resto do espaço disponível.
No resto deste guia irei considerar que o nome da partição de boot é `/dev/sda1` e o da partição raiz é `/dev/sda2`, <strong> você deve substituir pelos nomes das partições do seu sistema.</strong>

#### Formatar partições
{{< highlight bash >}}
mkfs.fat -F32 /dev/sda1 # Formata a partição de boot com Fat32
mkfs.btrfs /dev/sda2 # Formate a partição raiz com btrfs
{{</ highlight >}}

#### Criando subvolumes
Uma das principais razões para se utilizar btrfs é a facilidade para fazer o backup e restaurar o sistema, funcionalidade essa que só pode ser utilizada se tivermos subvolumes. Nessa estapa o nosso objetivo é criar os seguintes subvolumes:
| Subvolume | Descrição |
|-----------|-----------|
|@root|Raiz do sistema|
|@home|Pasta dos usuários|
|@var|(Opcional) será montada em /var|
|@tmp|(Opcional) será montada em /tmp|
|@snapshots| Aqui é onde será salvo os snapshots do sistema|
|@swap| Aqui será onde criaremos o nosso swapfile|

Com os seguintes comandos:
{{< highlight bash >}}
mount /dev/sda2 /mnt # Monta a partição para podermos criar os subvolumes
btrfs subvolume create /mnt/@root # Criar o subvolume @root
btrfs subvolume create /mnt/@home # Criar o subvolume @home
btrfs subvolume create /mnt/@var # Criar o subvolume @var
btrfs subvolume create /mnt/@tmp # Criar o subvolume @tmp
btrfs subvolume create /mnt/@snapshots # Criar o subvolume @snapshots
btrfs subvolume create /mnt/@swap # Criar o subvolume @swap
umount /mnt # Desmonta a partição para podermos utilizar os subvolumes que acabamos de criar
{{</ highlight >}}

Agora precisamos montar os subvolumes:

{{< highlight bash >}}
mount -o defaults,noatime,compress=zstd:2,subvol=@root /dev/sda2 /mnt # Monta a raiz do sistema
# Se o sistema não for efi mude boot/efi,... para boot...
mkdir -p /mnt/{boot/efi,var,home,tmp,.snapshots,.swap} # Cria todos os diretórios especificados recursivament
mount -o defaults,noatime,compress=zstd:2,subvol=@home /dev/sda2 /mnt/home # Monta a pasta do usuário
mount -o defaults,noatime,compress=zstd:2,subvol=@var /dev/sda2 /mnt/var
mount -o defaults,noatime,compress=zstd:2,subvol=@tmp /dev/sda2 /mnt/tmp
mount -o defaults,noatime,compress=zstd:2,subvol=@snapshots /dev/sda2 /mnt/.snapshots
mount -o defaults,noatime,compress=zstd:2,subvol=@swap /dev/sda2 /mnt/.swap
# Se o sistema não for efi mude /mnt/boot/efi para /mnt/boot
mount /dev/sda1 /mnt/boot/efi
{{</ highlight >}}

Explicação: `compress=zstd:2` define o algoritmo e o nível de compressão do sistema de arquivos, utilize `compress=none` se quiser optar por não utilizar isso. `noatime` sinaliza para o sistema não escrever no arquivo quando ele for lido isso pode melhorar a vida útil de ssds já que tem que fazer menos escritas mas pode fazer com que aplicações que dependam disso não funcionem direito.
Nesse passo, se o sistema for BIOS e não UEFI, mude todas as menções acima de boot/efi para boot.

### 3. Instalação
{{< highlight bash >}}
pacstrap /mnt base base-devel linux linux-firmware nano # ou vim depende de sua preferência
{{</ highlight >}}

### 4. Configurar o sistema
#### Configurar swapfile
{{< highlight bash >}}
truncate -s 0 /mnt/.swap/swapfile # Cria o swapfile com tamanho 0
chattr +C /mnt/.swap/swapfile # Desabilita COW no arquivo
btrfs property set /mnt/.swap/swapfile compression none # Força o arquivo a não ser compressado, o comando acima deve fazer com que esse não seja necessário, mas está aqui por via das dúvidas
fallocate -l 9G /mnt/.swap/swapfile # aloca 9G de espaço para o swapfile, esse tamanho só é necessário, na maior parte dos casos, se quisermos hibernar nossa máquina.
chmod 600 /mnt/.swap/swapfile # Define permissões de forma que apenas o usuario root possa acessar o arquivo
mkswap /mnt/.swap/swapfile # Cria swap no arquivo
swapon /mnt/.swap/swapfile # Monta o swap 
{{</ highlight >}}

#### Gerar fstab
fstab é o arquivo responsável por definir como as partições serão montadas durante a inicialização do sistema. Esse arquivo pode ser gerado de acordo com as partições montadas em um determinado diretório da seguinte forma:
{{< highlight bash >}}
genfstab -U /mnt >> /mnt/etc/fstab
{{</ highlight >}}

#### Chroot, mude a raiz para o sistema instalado
{{< highlight bash >}}
arch-chroot /mnt # Muda a raiz do sistema, ou seja, alterações aqui serão feitas no sistema instalado, até sair desse bash.
{{</ highlight >}}
A partir daqui estaremos configurando o sistema instalado, portanto tenha certeza de não sair do bash acima até que tenhamos terminado tudo.

#### Fuso Horário
{{< highlight bash >}}
# ln -sf /usr/share/zoneinfo/Região/Cidade /etc/localtime
ln -sf /usr/share/zoneinfo/America/Recife /etc/localtime # Define o fuso horário para o da cidade de Recife
hwclock --systohc # Gera /etc/adjtime
{{</ highlight >}}

#### Localização
Locales ou localizações são utilizados por programas e bibliotecas conscientes dos locales para definir como renderizar corretamente textos, valores, horário, etc...
Para gerar locales é preciso editar o arquivo /etc/locale.gen e descomentar as linhas com locales desejados, no nosso caso pt_BR.UTF-8 UTF-8 (e em alguns casos en_US.UTF-8 UTF-8 também já que já tive problemas por não ter esse locale gerado no sistema).
{{< highlight bash >}}
locale-gen # gera locales de linhas descomentados no arquivo /etc/locale.gen 
echo "LANG=pt_BR.UTF-8" > /etc/locale.conf # Escreve a lingua que vai ser utilizada pelo sistema no arquivo /etc/locale.conf
echo "KEYMAP=br-abnt2" > /etc/vconsole.conf # Define o mapa de teclas que vai ser utilizado pelos consoles virtuais do sistema 
{{</ highlight >}}

#### Rede
{{< highlight bash >}}
echo "meu-nome-de-host" > /etc/hostname # Define o nome do host
{{</ highlight >}}

#### Segurança
Altere a senha do usuário root
{{< highlight bash >}}
passwd # vai lhe ajudar a definir a senha do usuário atual (root)
{{</ highlight >}}

Adicione o seu usuário, afinal não se deve ficar utilizando o sistema como root
{{< highlight bash >}}
useradd -m -g users -G wheel,storage,power,audio -s /bin/bash seunomedeusuario
passwd seunomedeusuario # define a senha do seu usuário
{{</ highlight >}}

### 5. Configurar Inicialização do sistema
{{< highlight bash >}}
# Instalando um gerenciador de boot e dependencias necessárias
# efibootmgr só é necessário se estiver em um sistema EFI
pacman -S grub grub-btrfs efibootmgr dosfstools mtools ntfs-3g
# Se o sistema for EFI use o comando abaixo para installar o grub (gerenciador de boot)
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB 
# Se não, utilize:
# grub-install /dev/sda # ou qualquer que seja o nome do disco onde está a partição raiz do seu sistema
grub-mkconfig -o /boot/grub/grub.cfg # gera o arquivo de configuração de boot
{{</ highlight >}}

### 6. Instalar ambiente desktop Plasma e rede
{{< highlight bash >}}
pacman -S sddm plasma networkmanager # Instala ambiente desktop plasma, gerenciador de display e gerenciador de rede
systemctl enable sddm # habilita o sddm a ser inicializado junto com o sistema
systemctl enable NetworkManager # habilita o gerenciador de rede a ser inicializado junto com o sistema
{{</ highlight >}}

### 7. Finalização
Pronto. Você instalou o Arch Linux.
Agora:
{{< highlight bash >}}
exit # para sair do chroot
umount -a # desmonta todas as partições montadas
reboot # reinicia o sistema.
{{</ highlight >}}

### Referências

[Copy-on-Write](https://pt.wikipedia.org/wiki/C%C3%B3pia_em_grava%C3%A7%C3%A3o)
[Instalação Archlinux com BTRFS (medium.com)](https://medium.com/@fredlins/instala%C3%A7%C3%A3o-do-arch-linux-com-btrfs-grub-btrfs-snapper-snapper-gui-dc6eb1371f43#a2d7)
[Instalação Archlinux (archlinux.org)](https://wiki.archlinux.org/index.php/Installation_guide_(Portugu%C3%AAs)#Pr%C3%A9-instala%C3%A7%C3%A3o)
[Instalando btrfs com pop_os](https://mutschler.eu/linux/install-guides/pop-os-btrfs/)