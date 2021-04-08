---
title: "Archlinux Installation With Btrfs"

date: 2021-04-08T11:46:29-03:00
url: /archlinux-installation-with-btrfs/
image: images/2021-thumbs/archlinux-installation-with-btrfs.png
categories:
  - Linux
  - Btrfs
tags:
  - Archlinux
draft: false
---
## Instalação do sistema (Em construção)
Neste tutorial assumo que você já tenha uma mídia de instalação pronta e sabe como inicializar o sistema por ela.
<!--more-->
### Configuração do ambiente
#### Mapeamento de teclas
O mapa de teclas padrão do console é US, mas ele pode ser facilmente alterado de com o comando `loadkeys`. Para alterar o mapa de teclas para o padrão brasileiro podemos utilizar o seguinte comando:
{{< highlight bash >}}
loadkeys br-abnt2
{{</ highlight >}}

Para saber quais mapas de teclas estão disponíveis no seu sistema:
{{< highlight bash >}}
localectl list-keymaps
{{</ highlight >}}

### Partições do disco
#### Listar discos
O comando `lsblk` retornará os discos e partições com dados extras, como tamanho e ponto de montagem. 
#### Criar partições
Nesse ponto temos duas opções, que eu conheço, de comandos que podem ser utilizados: `fdisk` e `cfdisk`. Dentre essas duas opções `cfdisk` é mais amigável a novatos, já que é uma aplicação com interface gráfica.
Uma vez que tenha escolhido o disco que irá utilizar para instalar o archlinux é só utilizar um dos comandos acima seguido pelo nome do disco. Ex.:
{{< highlight bash >}}
cfdisk /dev/sda
{{</ highlight >}}

* test
 - test
* test