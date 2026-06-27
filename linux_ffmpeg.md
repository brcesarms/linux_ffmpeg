Este comando utiliza o FFmpeg para realizar uma transcodificação de vídeo otimizada, aproveitando a aceleração de hardware da placa NVIDIA RTX 5060.

```bash
# sem perda de qualidade
ffmpeg -i ENTRADA.mp4 -c:v av1_nvenc -preset p7 -cq 26 -filter:v fps=60 -c:a libopus -b:a 128k SAIDA.mkv
 
# menos parametros
ffmpeg -i ENTRADA.mp4 -c:v av1_nvenc -c:a libopus SAIDA.mkv
 
```

#### O que cada parte do comando faz:
* `-i entrada.mp4`: Especifica o arquivo de vídeo original. Substitua `entrada.mp4` pelo nome exato do seu vídeo.

* `-c:v av1_nvenc`: O "pulo do gato". Diz ao FFmpeg para usar o codificador AV1 acelerado pela sua placa de vídeo NVIDIA.

* `-preset p7`: Uma predefinição nativa da NVIDIA que oferece um excelente balanço entre qualidade de imagem e eficiência.

* `-cq 26`: Define uma "Qualidade Constante" (Constant Quality). O valor 30 é um bom ponto de partida para AV1. Se achar que o vídeo perdeu qualidade, diminua esse valor (ex: 26 ou 28). Se o arquivo ficar muito grande, aumente-o (ex: `34`).

* `-filter:v fps=60`: Garante que o vídeo de saída tenha exatamente 60 quadros por segundo.

* `-c:a libopus`: Converte a faixa de áudio para o codec Opus.

* `-b:a 128k`: Define a taxa de bits do áudio. O Opus é incrivelmente eficiente, então 128 kbps geralmente entrega uma qualidade de som transparente (indistinguível da fonte original) para áudio estéreo.

* `saida.mkv`: O nome e o contêiner (MKV) do arquivo final.

&nbsp;

---


```bash
# com perda de qualidade
ffmpeg -i ENTRADA.mp4 -c:v av1_nvenc -preset p6 -cq 45 -vf "scale=-1:720,fps=24" -c:a libopus -b:a 32k -ac 1 SAIDA.mkv
 
```

1. Reduzir a Resolução (Downscale)
Esta é a forma que mais economiza espaço. Se o seu vídeo original foi gravado em 1080p (Full HD) ou 4K, reduzi-lo para 720p (HD) cortará o tamanho do arquivo drasticamente. Em telas de celular ou janelas menores do seu Fedora 44, a diferença de nitidez costuma ser aceitável.

* Adicione o filtro: `-vf scale=-1:720`.

2. Elevar a Compressão ao Extremo (CQ 40+)
Como você já aprendeu, quanto maior o número do -cq, menor será o tamanho final do arquivo. Você pode testar valores extremos como 40, 45 ou até 50. A imagem ficará visivelmente com blocos (pixelada) nas partes de muito movimento, mas o tamanho em disco despencará.

* Altere o parâmetro para: `-cq 45`.

3. Áudio em Mono e Bitrate Mínimo
O codec Opus é absurdamente eficiente. Você pode baixar a taxa de bits para 48k ou até 32k. Além disso, se o áudio for focado em conversas e vozes (sem necessidade de imersão estéreo de músicas), você pode juntar os canais esquerdo e direito em um só (Mono).

* Use os parâmetros: `-c:a libopus -b:a 32k -ac 1`.

4. Forçar uma Taxa de Quadros (FPS) Menor
Forçar 60 fps em uma fonte de menor fluidez apenas obriga o FFmpeg a desenhar e salvar quadros duplicados. Para economizar o máximo de espaço, você pode forçar o vídeo inteiro a rodar a 24 quadros por segundo (o padrão da indústria do cinema). O movimento ficará com um aspecto menos fluido, mas você reduzirá enormemente a quantidade de imagens processadas e salvas a cada segundo.

* Use o filtro: `-filter:v fps=24`.

&nbsp;

---

#### ⚠️ Importante para o Fedora 44
Para que a aceleração por hardware da NVIDIA funcione perfeitamente no Fedora, certifique-se de que dois requisitos estejam atendidos:

1. Drivers Proprietários da NVIDIA: Você precisa estar com os drivers oficiais instalados (geralmente instalados via loja de aplicativos do GNOME/KDE ou via linha de comando pelo repositório `rpmfusion-nonfree`).

```bash
sudo dnf -y update
 
```

```bash
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

 
```

```bash
# reinicie o sistema
sudo dnf swap ffmpeg-free ffmpeg --allowerasing
 
```

Detalhando o comando:

* `sudo`: Executa a ação com privilégios de superusuário (administrador / root).


* `dnf`: É o nome do gerenciador de pacotes padrão do Fedora.

* `swap`: É a instrução mágica aqui. Ela manda o sistema "trocar" um pacote por outro em uma única operação, evitando conflitos ou quebras.

* `ffmpeg-free`: É o nome do pacote limitado que será removido do seu sistema.


* `ffmpeg`: É o nome do pacote completo que será instalado no lugar (obtido a partir do repositório RPM Fusion).

* `--allowerasing`: Dá permissão explícita ao dnf para apagar pacotes que estejam causando conflito (neste caso, o ffmpeg-free antigo) para permitir a entrada da versão nova.

Rodar este comando é um passo obrigatório para garantir que todos os codecs e a aceleração da sua placa de vídeo NVIDIA funcionem perfeitamente.

&nbsp;

---
```bash
sudo dnf group upgrade multimedia
 
```

Detalhando o comando:

* `sudo dnf`: Roda o gerenciador de pacotes do Fedora com privilégios de administrador.

* `group upgrade`: É uma instrução que diz ao sistema para atualizar um "grupo" inteiro de softwares interligados, em vez de um aplicativo isolado.

* `multimedia`: É o nome desse grupo. Ele engloba ferramentas como o próprio FFmpeg, reprodutores de vídeo, codecs de áudio e plugins essenciais (como os do GStreamer).

Para o seu caso específico, lidando com conversões pesadas de AV1 e MKV, rodar isso garante que todo o ecossistema de mídia do seu Fedora esteja na versão mais recente e otimizada.

&nbsp;

---

```bash
sudo dnf group upgrade core
 
```

Detalhando o comando:

* `sudo dnf`: Roda o gerenciador de pacotes do Fedora com privilégios de administrador.

* `group upgrade`: É uma instrução que diz ao sistema para atualizar um "grupo" inteiro de softwares interligados, em vez de um aplicativo isolado.

* `core`: É o nome desse grupo base. Ele não mexe com interface gráfica ou players de vídeo, mas sim com os componentes críticos e de baixo nível do sistema operacional (como utilitários do kernel, inicialização e ferramentas básicas do terminal).
