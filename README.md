# Driver Mínimo para GMAX3412

O GMAX3412 requer um gatilho externo para gerar o quadro, portanto ele não consegue operar por conta própria como a maioria dos sensores de câmera MIPI. Como resultado, este driver está sem controles críticos do V4L2, como `V4L2_CID_VBLANK`, `V4L2_CID_HBLANK` e `V4L2_CID_EXPOSURE`. A expectativa é que os usuários gerem os pulsos externamente por outros meios.

A forma como este driver está configurado atualmente é com exposição externa, o que significa que a exposição e a taxa de quadros são determinadas pela frequência de um pulso externo e pela duração do nível alto. O driver não faz nada com os controles `V4L2_CID_VBLANK`, `V4L2_CID_HBLANK` e `V4L2_CID_EXPOSURE`.

Mesmo assim, ele continua expondo esses controles para a camada superior porque alguns aplicativos (como `rpicam` e `libcamera`) exigem esses controles mínimos.

O driver exige MIPI de 4 vias, pois não vi uma forma de usar 2 ou 1 via a partir do datasheet vazado (ou "leaked").
Além disso, ele suporta apenas o modo 12-bit 4K (4096x3072). Se alguém tiver uma versão mais atualizada do datasheet, sinta-se à vontade para abrir uma issue e enviar o arquivo.

No geral, isso é uma extensão do meu driver mínimo básico para `gmax4002`, apenas adaptado para o `gmax3412`.

## Plataforma de trabalho
O código foi testado no Raspberry Pi 5 com o Analog Discovery 2 ou um MCU como gerador de pulsos. Consulte o repositório da placa da câmera [aqui](https://github.com/will127534/GlobalEye) para obter o código do MCU e detalhes da placa. O suporte à libcamera foi adicionado ao meu fork da libcamera [aqui](https://github.com/will127534/libcamera).

<img width="1280" alt="image" src="https://github.com/user-attachments/assets/53eb4a42-8ea5-4f12-b764-6d9b37767cd4" />
<img width="1280" alt="image" src="https://github.com/user-attachments/assets/5b95bd54-53f1-42e3-ba69-38ca70e62af9" />

Veja em ação aqui: [YouTube](https://www.youtube.com/watch?v=J_Mvx6Y6Drg).
A placa foi testada com Raspberry Pi 5.

O sinal de trigger se parece com isto:
<img width="1280" alt="image" src="https://github.com/user-attachments/assets/56f11915-6237-4873-89a1-e6653319db5b" />
Amarelo é o TEXP, e o azul é a saída TDIG do sensor mostrando o "Frame Overhead Time".

## Pré-requisitos

Antes de iniciar o processo de instalação, certifique-se de que os seguintes pré-requisitos sejam atendidos:

- **Versão do kernel**: você deve estar executando um kernel Linux 6.12 ou superior. Você pode verificar a versão do kernel executando `uname -r` no terminal.

- **Ferramentas de desenvolvimento**: ferramentas essenciais como `gcc`, `dkms` e `linux-headers` são necessárias para compilar um módulo do kernel. Se ainda não estiverem instaladas, elas podem ser instaladas com o gerenciador de pacotes usando o seguinte comando:

   ```bash
   sudo apt install linux-headers dkms git
   ```

## Etapas de instalação

### Configurando as ferramentas

Primeiro, instale as ferramentas necessárias (`linux-headers`, `dkms` e `git`) se ainda não tiver feito isso:

```bash
sudo apt install linux-headers dkms git
```

### Obtendo o código-fonte

Clone o repositório para sua máquina local e navegue até o diretório clonado:

```bash
git clone https://github.com/will127534/gmax3412-v4l2-driver.git
cd gmax3412-v4l2-driver/
```

### Compilando e instalando o driver do kernel

Para compilar e instalar o driver do kernel, execute o script de instalação fornecido:

```bash
./setup.sh
```

### Atualizando a configuração de boot

Edite o arquivo de configuração do boot com o seguinte comando:

```bash
sudo nano /boot/config.txt
```

No editor aberto, localize a linha contendo `camera_auto_detect` e altere seu valor para `0`. Em seguida, adicione a linha `dtoverlay=gmax3412`. Ficará assim:

```
camera_auto_detect=0
dtoverlay=gmax3412
```

Depois de fazer essas alterações, salve o arquivo e saia do editor.

Lembre-se de reiniciar o sistema para que as alterações entrem em vigor.

## Opções do dtoverlay

### cam0

Se a câmera estiver conectada na porta `cam0`, acrescente o `dtoverlay` com `,cam0`, assim:

```
camera_auto_detect=0
dtoverlay=gmax3412,cam0
```

### always-on

Se você quiser manter a energia da câmera sempre ligada (útil para depuração de problemas de hardware; especificamente, isso define `CAM_GPIO` como alto o tempo todo), acrescente o `dtoverlay` com `,always-on`, assim:

```
camera_auto_detect=0
dtoverlay=gmax3412,always-on
```
