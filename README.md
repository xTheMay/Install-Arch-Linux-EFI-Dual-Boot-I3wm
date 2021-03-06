# Install-Arch-Linux-EFI-Dual-Boot-I3wm
## Instalação e configuração do Arch Linux UEFI/GPT e Dual Boot com Windows

 _Ao bootar a mídia de instalação do Arch, você estará logado como # (root) em automatic login._<br>
 _*Será utilizado o editor de texto nano ao decorrer do tutorial (para salvar: "ctrl+o", sair: "ctrl+x"), fica a seu critério qual editor utilizar!_<br><br>

0- Carregar o layout do teclado
> loadkeys br-abnt2

1- Testar a Conexão
> ping -c 3 www.google.com

**Caso precise conectar sua internet**<br>
*Para rede cabeada:
> dhcpcd

*Para rede wireless:
> wifi-menu

2- Listar partições existentes:
> fdisk -l

**3- Configurar as partições no disco ( _fica a seu critério, um exemplo para 50gb:_ )**<br>
**Você será responsável pelos seus dados ,perdas poderão ocorrer caso não seja cuidadoso.**<br>
_*Aqui entra o Dual Boot_
> cgdisk /dev/sda 
>>Observação: _‘sda’ (Primeiro disco rígido SATA) ou ‘sdb’ (Segundo disco rígido SATA) ou 'nvme0n1' (Primeiro SSD M.2)_

Ex de partições:<br> sda1 150MB Recovery<br>
    sda2 512MB Partição EFI<br>
    sda3 700GB Partição do Windows<br>
sda4 * 48G /<br>
sda5 2G swap<br><br>
Escrever/Write — Sim/Yes - Quit/Sair<br><br>

Então nesse exemplo ficou: **/ (root)** no sda4, **swap** no sda5 e já existia uma partição EFI que o windows criou no sda2 (que vai ser usada posteriormente para o /boot)<br>

4- Formatar as partições com o sistema de arquivos ext4:<br>
**_*A partir deste passo, verifique os números dos sda, pois estou utilizando os do exemplo acima!_**
> mkfs.ext4 /dev/sda4

5- Formatar e ativar swap:
>mkswap /dev/sda5
>>swapon /dev/sda5

**6- Montar partições** 
> mount /dev/sda4 /mnt

_*Partições adicionais são montadas da mesma maneira._

7- Para o /boot:
>mkdir /mnt/boot
>>mount /dev/sda2 /mnt/boot

8- Visualizar o particionamento atual:
>lsblk /dev/sda

**9- Instalar o sistema base**
>pacstrap /mnt base base-devel
 
10- Gerar arquivo fstab (FSTAB: File System Table):
>genfstab -U -p /mnt/ >> /mnt/etc/fstab

11- Utilizar o ambiente chroot (Chroot: Change Root)
> arch-chroot /mnt /bin/bash

12- Configurar Localização:
> nano /etc/locale.gen
>>**descomentar essa linha:**
>>> pt_BR.UTF-8 UTF-8

13- Gerar arquivo de localização
>locale-gen

14- Criar o arquivo de configuração de idioma novamente:
>echo LANG=pt_BR.UTF-8 > /etc/locale.conf
>>export LANG=pt_BR.UTF-8

15- Para que tais configurações fiquem guardadas, edite o arquivo vconsole.conf:
>nano /etc/vconsole.conf

Escreva e depois salve:
>KEYMAP=br-abnt2

16- Configurar o fuso-horário
>ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime

17- Sincronizar o relógio do hardware com o do sistema
>hwclock --systohc --utc

18- Configurar o repositório para x64
>nano /etc/pacman.conf

_**Descomentar essas linhas e depois salvar:**_
>[multilib]
>>include = /etc/pacman.d/mirrorlist

19- Sincronizar os repositórios
>pacman -Sy

20- Definir um nome para o SO:
> echo nomedopc > /etc/hostname

21- Adicionando entrada em hosts
>nano /etc/hosts

Deixe parecido com isso:
>127.0.0.1 localhost.localdomain localhost
>>::1 localhost.localdomain localhost
>>>127.0.1.1 nomedopc.localdomain nomedopc

22- Baixar Sudo e Grub
> pacman -S sudo grub efibootmgr os-prober

**23- Instalar o GRUB**
>grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --recheck

24- Criar um ambiente ramdisk inicial
> mkinitcpio -p linux

25- Gerar arquivo de configuração do GRUB
> grub-mkconfig -o /boot/grub/grub.cfg

26- Criar usuario e definir senha
>useradd -m -g users -G wheel,storage,power -s /bin/bash seuuser 
>>passwd seuuser

27- Definir a senha de root
>passwd root

28- Editar o arquivo sudoers:
>nano /etc/sudoers

_**Descomente a opção e salve**_
> %wheel ALL=(ALL) ALL

**29- Instalar componentes do Wi-Fi.**
>pacman -S wpa_supplicant networkmanager net-tools
>>systemctl enable NetworkManager

_**30- Caso esteja instalando em um notebook, o seguinte comando para drivers de touchpad:**_
>pacman -S xf86-input-synaptics

**Pronto para reiniciar o sistema!**
>exit
>>umount /mnt/boot
>>>umount /mnt

>reboot

**O sistema base está instalado!!!**<br>
_A partir daqui é possível se virar sozinho, darei uma base de onde começar, com a interface i3 e um gerenciador de login básico, o ideal é pesquisar e personalizar como quiser!_<br>
<br>0- Interface básica de configuração de rede:
>nmtui

1- Para ter algum tipo de interface/GUI, execute o seguinte comando:
>sudo pacman -S xorg-server xorg-xinit xorg-apps gvfs-mtp sshfs xterm

2- Verifique sua placa gráfica
>  lspci | grep VGA

Algumas opções de instalação (no meu caso usei o da Intel):<br>
 pacman -S virtualbox-guest-utils — para o virtualbox<br>
pacman -S xf86-video-amdgpu — para placas amd-radeon<br>
 pacman -S xf86-video-nouveau — para placa de vídeo Nvidia)<br>
_pacman -S xf86-video-intel — para drivers da intel_<br>

3- Instalar utilitários de som
> sudo pacman -S pavucontrol alsa-firmware alsa-utils alsa-plugins pulseaudio-alsa pulseaudio

4- Instalar uma interface (no meu caso, o i3)
> sudo pacman -S i3

5- Instalar um gerenciador de login (no meu caso, o lightDM)
> sudo pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
>> sudo systemctl enable lightdm

6- Deixar configuração do teclado salva no ambiente X:
> sudo localectl set-x11-keymap br abnt2

7 - Criar e colocar pastas padrões dos usuários
> pacman -S xdg-user-dirs
>> xdg-user-dirs-update

>reboot

**A partir daqui é mais do que suficiente para explorar a distro Arch Linux com o i3!!!**
*Caso não apareça o Windows no grub, execute novamente, só que agora logado no arch e utilizando o terminal, o seguinte comando:
> grub-mkconfig -o /boot/grub/grub.cfg

*O os-prober irá detectar o windows e aparecerá novamente no grub!
