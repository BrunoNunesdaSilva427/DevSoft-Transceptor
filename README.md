# 📻 DevSoft Transceptor

**Rádio PX/CB virtual com voz em tempo real, direto no navegador.**
Um transceptor com visual retrô (painel com knobs, chaves, botão power, medidor e display de canal) que conecta pessoas em salas por canal usando WebRTC peer-to-peer, sem servidor próprio e sem build — é um único arquivo `index.html`.

> Autor: **Bruno Nunes da Silva** · DevSoft AI

---

## ✨ Funcionalidades

### Rádio

- **Voz push-to-talk (PTT)** em tempo real entre os participantes da sala, com bloqueio de TX ("travar TX") para toque.
- **Botão power com LED**: liga e desliga o rádio. Desligado, o áudio fica mudo, o display, o medidor e o LED TX/RX apagam e a transmissão (inclusive a da ponte JARVIS) é bloqueada. O estado fica salvo no navegador.
- **99 canais em 3 bandas** (`LOW`, `MID`, `HI`), com frequência exibida no estilo faixa de 27 MHz (ex.: `26.965 MHz` no canal 1 da banda MID).
- **Salas por canal**: crie uma sala no canal sintonizado ou entre em uma sala aberta listada no lobby. Detecção de canal ocupado.
- **Salas privadas com senha** 🔒: senha de no mínimo 8 caracteres, digitada em diálogo mascarado, que pode ser **permanente** ou **temporária** (5 min, 15 min, 30 min ou 1 hora). Veja [Salas privadas](#-salas-privadas).
- **Convite por link** (`#sala=M12&n=Nome`) com Web Share API ou cópia para a área de transferência.
- **Modos de recepção**: `CW`, `FM`, `AM`, `USB` e `LSB`, cada um com filtros e nível de ruído próprios.
- **Processamento de áudio realista (Web Audio API)**: squelch, ganho de RF, noise blanker (NB), ruído de fundo, tom de CW, side-tone e beeps de canal ocupado.
- **Mapa das estações** (Leaflet + OpenStreetMap) com a localização aproximada (arredondada a ~1 km) de quem está na sala, se o usuário permitir.
- **Camadas do mapa**: aeronaves em tempo real (Flightradar24, com adsb.lol, airplanes.live e outras fontes de reserva) e a posição da ISS com trilha e previsão. Veja [Aviões e ISS](#-aviões-e-iss).
- **Transcrição de fala** via Web Speech API (Chrome/Edge), enviada como texto para a sala, com exportação em **JSON** e **CSV**.
- **Ponte JARVIS / IA**: aceita áudio e texto de um agente externo e transmite na sala; expõe a API global `window.RA` e integra com WebView2 (`chrome.webview`) para apps desktop.
- **Design do rádio** personalizável (salvo em `localStorage`): estilos Stryker SR-497HPC, SR-955HP, 148 GTL e Clássico, 12 predefinições de cores no estilo de rádios PX e 7 acabamentos metalizados. Veja [Design do rádio](#-design-do-rádio).
- **Layout vertical para celulares**: abaixo de 820 px o microfone PTT, um painel de controles rápidos (volume, squelch, ganho do mic, ganho RF e canal −/+), o rádio, a transcrição, o mapa e a lista de estações se empilham em largura total.
- Tela de abertura animada, relógio no cabeçalho e botão de reset dos controles.

### Recursos extras

Painéis recolhíveis abaixo da transcrição, no mesmo tema do app. Veja os detalhes em [Recursos extras em detalhe](#-recursos-extras-em-detalhe).

- **💬 Chat da sala** com envio de **posição** (📍) que abre direto no mapa; cifrado em salas privadas.
- **📣 Chamada seletiva**: chama uma estação específica pelo indicativo, com toque de alerta.
- **⏺ Gravações**: grava as últimas 10 transmissões (recebidas e suas), com play, download e **repetir última**.
- **🎚 Redução de ruído** opcional no áudio recebido.
- **🔔 Alertas** (SOS, chamada, chat e sala nova), **app instalável (PWA)** e **tela sempre ligada** enquanto estiver numa sala.
- **🗺 Trilhas no mapa** e popup com distância e rumo de cada estação.
- **⌨ Atalhos** de teclado e botões de fones Bluetooth (play/pause vira PTT).
- **🛰 Avisos do canal 9**: ISS passando perto e boletim do tempo na transcrição.
- **🌐 Tradução** das falas para 6 idiomas e **leitura em voz alta**.
- **👑 Moderação**: dono da sala pode mutar, expulsar e pedir prioridade; modo "só escuta".
- **✔ Verificação de identidade** por assinatura digital, com selo na lista de estações.
- **📒 Diário de bordo** (log de contatos estilo radioamador) com estatísticas, exportação **ADIF/CSV** e **cartão QSL** em PNG.
- **★ Favoritos** de salas e estações, com backup e importação.
- **🌉 Ponte de texto** entre duas salas.
- **🔍 Scan** de salas ativas.

---

## 🧱 Como funciona

```
 ┌────────────┐   presença / sinalização / transcrição    ┌────────────────┐
 │  Browser A │ ◄──────────── MQTT (WebSocket) ─────────► │ Brokers MQTT   │
 └─────┬──────┘                                           │ públicos       │
       │                                                  └────────────────┘
       │  áudio (WebRTC, malha P2P — STUN/TURN quando necessário)
       ▼
 ┌────────────┐
 │  Browser B │
 └────────────┘
```

| Camada | Tecnologia |
|---|---|
| Áudio / UI do rádio | Web Audio API, HTML, CSS, JavaScript puro |
| Voz | WebRTC (`RTCPeerConnection`), malha P2P entre membros da sala |
| Sinalização, presença, transcrição, chat e moderação | MQTT 5 sobre WebSocket (`mqtt.js`), com vários brokers públicos em paralelo (EMQX, HiveMQ, Mosquitto, Eclipse, shiftr.io) |
| Salas privadas | Web Crypto API: PBKDF2-SHA-256, ECDH P-256, AES-GCM e HMAC-SHA-256 |
| Identidade das estações | Web Crypto API: ECDSA P-256 com fixação na primeira vez (TOFU) |
| Gravação e redução de ruído | `MediaRecorder` e nós de filtro/compressor da Web Audio API |
| Fones, notificações e tela ligada | Media Session API, Notifications API e Screen Wake Lock API |
| Fallback de conexão | PeerJS (modo hub/cliente) |
| NAT traversal | STUN (Google, Twilio, Cloudflare) e TURN (Metered) |
| Mapa | Leaflet 1.9 + tiles do OpenStreetMap |
| Aviões e ISS | Flightradar24, adsb.lol, adsb.fi, adsb.one, airplanes.live, OpenSky e wheretheiss.at |
| Tempo (canal 9) | Open-Meteo |
| Tradução | MyMemory (API gratuita) |
| Reconhecimento e síntese de fala | Web Speech API (`SpeechRecognition` e `speechSynthesis`) |

Dependências são carregadas por CDN (jsDelivr / cdnjs, com fallback para unpkg). Não há `npm install`, bundler nem backend.

---

## 🚀 Como usar

### Rodando localmente

Microfone, geolocalização e salas privadas exigem **contexto seguro** (`https://` ou `localhost`). Abrir o arquivo direto (`file://`) funciona para testar a interface, mas o link de convite só serve para outras pessoas se o app estiver publicado.

```bash
git clone https://github.com/<seu-usuario>/<seu-repo>.git
cd <seu-repo>

python3 -m http.server 8080
# depois abra http://localhost:8080
```

### Publicando

Por ser um arquivo estático, funciona em GitHub Pages, Netlify, Vercel, Cloudflare Pages ou qualquer hospedagem estática com HTTPS.

### Entrando em uma sala

1. Informe seu **indicativo** (mín. 3 caracteres, ex.: `PY2ABC`) e, opcionalmente, seu nome.
2. Gire o seletor de canal / banda ou escolha uma sala aberta no lobby.
3. Permita o acesso ao microfone.
4. Confirme que o botão **POWER** está aceso, segure o botão **MIC** para transmitir (ou use **TRAVAR TX** no celular).
5. Use **🔗 CONVIDAR** para chamar alguém direto para a sua sala.

### Usando outros brokers MQTT

Passe um ou mais brokers pela query string:

```
https://seu-site/?broker=wss://meu-broker:8084/mqtt&broker=wss://outro-broker/mqtt
```

---

## 🧩 Recursos extras em detalhe

### ⌨ Atalhos

| Tecla / botão | Ação |
|---|---|
| `ESPAÇO` | PTT (segurar para falar) |
| `[` e `]` | Canal anterior / próximo |
| `R` | Repetir a última transmissão gravada |
| `S` | Liga/desliga o scan |
| Play/pause do fone Bluetooth | Alterna o PTT |
| Próxima/anterior do fone Bluetooth | Canal +1 / −1 |

Os atalhos de letra só funcionam quando nenhum campo de texto está em foco. Navegadores não permitem capturar os botões de volume do celular, por isso o PTT por hardware usa os botões de mídia do fone.

### 💬 Chat e chamada seletiva

- Mensagens de até 300 caracteres para a sala. O botão **📍 POSIÇÃO** envia sua localização aproximada e quem recebe pode clicar em **ver no mapa**.
- Em **salas privadas**, texto e posição do chat vão cifrados com a chave da sala (AES-GCM). Em salas abertas trafegam em texto pelos brokers públicos.
- **📣 CHAMAR**: escolha uma estação na lista e ela recebe um toque de alerta e um aviso no display, sem incomodar o resto da sala.

### ⏺ Gravações

- Cada transmissão (recebida ou sua) vira um arquivo de áudio; ficam as **últimas 10** em memória, com indicativo, horário e duração. Elas somem ao recarregar a página.
- Ações: **▶** ouvir, **⬇** baixar (`.webm`) e **↺ REPETIR ÚLTIMA** (ou a tecla `R`).
- Trechos com menos de 0,8 s são descartados e cada gravação é cortada em 90 s. A gravação pode ser desligada no checkbox **GRAVAR**.

### 🎚 Áudio, alertas e instalação

- **Redução de ruído**: filtro passa-alta/passa-baixa com compressor aplicado ao áudio recebido, antes do squelch.
- **Alertas**: notificações do sistema para SOS, chamada seletiva, chat e abertura de sala nova. Só são exibidas com a aba **em segundo plano** e depois que você permitir notificações.
- **App instalável**: o app declara um manifesto e exibe o botão **📲 INSTALAR APP** nos navegadores que oferecem a instalação. Não há service worker, então o app **não funciona offline**.
- **Tela ligada**: enquanto você está numa sala, o app pede ao navegador para não apagar a tela (Wake Lock).

### 🗺 Trilhas e popups no mapa

Cada estação com localização deixa uma trilha pontilhada com os últimos 60 pontos. Clicar no marcador mostra o indicativo, a distância em km e o rumo (ex.: `NE 045°`) em relação à sua posição.

### 🛰 Avisos do canal 9

Com a sala no **canal 9**, o rádio ligado, o checkbox **AVISOS CANAL 9** marcado e sua localização definida:

- Aviso quando a **ISS** passa a menos de 1.500 km de você (no máximo um a cada 25 minutos).
- **Boletim do tempo** (temperatura, vento e chuva) ao entrar e a cada 30 minutos, com destaque de alerta para chuva, vento forte ou tempestade.

Os avisos aparecem só na **sua** transcrição, como `CH9-AVISOS`, para que a sala não receba o mesmo aviso repetido por todos.

### 🌐 Tradução e voz alta

- Escolha o idioma em **TRADUZIR PARA** (inglês, espanhol, português, francês, italiano ou alemão). A tradução de cada fala aparece abaixo do texto original, com detecção automática do idioma de origem.
- **LER EM VOZ ALTA** fala as mensagens recebidas com a síntese de voz do navegador, no idioma escolhido. As suas próprias falas não são lidas.

### 👑 Moderação

- O **dono da sala** é quem entrou primeiro. Ele pode **mutar/liberar** e **expulsar** estações e usar **⚡ PRIORIDADE 15s**, que interrompe quem está transmitindo e segura o canal por 15 segundos.
- Qualquer participante pode ativar **SÓ ESCUTA** para bloquear a própria transmissão.
- As regras são aplicadas pelo app de cada participante, sem servidor central. Em sala aberta, um participante modificado poderia ignorá-las ou forjar comandos.

### ✔ Verificação de identidade

Cada instalação gera um par de chaves ECDSA P-256 guardado no navegador e assina o indicativo a cada 45 segundos. Os outros participantes conferem a assinatura e mostram na lista de estações:

| Selo | Significado |
|---|---|
| ✔ VERIFICADA | O indicativo já foi visto antes com a mesma chave |
| 🔑 NOVA | Primeira vez que esse indicativo é visto; a chave fica fixada |
| ⚠ IDENTIDADE DIFERENTE | O indicativo apareceu com outra chave: pode ser outra pessoa usando o mesmo indicativo |

É um modelo de confiança na primeira vez (TOFU): não há autoridade central. Quem limpa os dados do navegador gera uma chave nova e aparecerá como ⚠ para quem já o conhecia.

### 📒 Diário de bordo, ADIF e QSL

- Estatísticas: tempo de **TX** e **RX**, estações contatadas e total de contatos.
- Um contato é registrado automaticamente quando uma estação fala por mais de 1,5 s; a mesma estação não é registrada de novo em menos de 5 minutos. Ficam guardados os 500 mais recentes.
- Cada registro tem data, indicativo, frequência, banda, modo, sala e RST fixo `59`.
- **⬇ ADIF** gera o arquivo `.adi` (banda 11 m) para importar em programas de log de radioamador; **⬇ CSV** gera planilha.
- **🪪 QSL** cria um cartão em PNG, no visual do app, para compartilhar. Em navegadores com compartilhamento de arquivos abre o menu de compartilhar; nos demais, baixa a imagem.

### ★ Favoritos

- **★ SALA ATUAL** e **★ ESTAÇÃO** salvam salas e indicativos. O lobby também mostra o cartão **FAVORITOS** com **ENTRAR** direto e indicação 🟢/⚪ de quem está online.
- **⬇ BACKUP** exporta favoritos, diário e identidades em JSON; **⬆ IMPORTAR** mescla favoritos de um backup.

### 🌉 Ponte de texto

Marque **PONTE DE TEXTO PARA**, escolha banda e canal de destino, e o chat e a transcrição da sua sala passam a ser copiados para lá. Na sala de destino a origem aparece como `INDICATIVO@canal`.

- Leva **somente texto**, não áudio.
- Funciona apenas entre **salas abertas** e enquanto você estiver na sala de origem.
- A cópia não é repetida em sentido contrário, o que evita laços entre duas pontes.

### 🔍 Scan

Com o scan ligado, o rádio percorre as **salas abertas** que tenham outras pessoas, a cada poucos segundos. Quando detecta atividade, fica parado e só retoma 6 segundos depois do silêncio. O scan desliga ao sair da sala.

---

## 🔐 Salas privadas

Ao criar uma sala (pelo formulário do lobby ou pelo botão **CRIAR SALA AQUI**) você pode definir uma senha. Deixar em branco cria uma sala aberta.

- **Senha**: mínimo de 8 caracteres, digitada em campos mascarados, com confirmação.
- **Validade**: *Permanente* (vale enquanto a sala existir) ou *temporária* (5 min, 15 min, 30 min ou 1 hora). Depois do prazo, **ninguém novo consegue entrar** com aquela senha; quem já está na sala continua. Na lista, a sala aparece como «senha expirada» com o botão **🔒 FECHADA**.
- **Como funciona**: a sala tem uma chave aleatória de 256 bits, e a senha serve apenas como porta de entrada. A presença pública não contém salt nem verificador da senha, só um identificador dessa chave, que não permite adivinhar a senha. Quem quer entrar pede a chave a um membro online por um handshake cifrado com ECDH; o membro só a entrega se a prova da senha (PBKDF2 com 600 mil iterações) for válida. Com a chave, o áudio de sinalização é autenticado e as transcrições e o chat são cifrados com AES-GCM.
- **Proteções**: cada membro limita tentativas (3 erros bloqueiam o solicitante por 2 minutos, com teto global de falhas).
- **Sala some quando esvazia**: como a chave só existe na memória dos membros, uma sala privada precisa de pelo menos um membro online para aceitar novas entradas.
- **Recursos extras**: em salas privadas a ponte de texto fica desligada e o scan ignora a sala.
- **Limite conhecido**: o protocolo não é um PAKE. Um atacante ativo que consiga responder a um pedido de entrada antes de um membro legítimo pode tentar adivinhar offline a senha de quem está entrando. Use senhas longas e, quando possível, temporárias.

---

## ✈ Aviões e ISS

O mapa tem dois botões de camada: **✈ AVIÕES** e **🛰 ISS**, mais um atalho **↗ FLIGHTRADAR24** que abre o site na região atual.

- Os aviões aparecem a partir do **zoom 6**. A busca tenta primeiro o feed do **Flightradar24** e, se falhar, cai para adsb.lol, adsb.fi, adsb.one, airplanes.live e OpenSky.
- Cada fonte é tentada direto e, se o navegador bloquear (CORS, extensão, firewall, VPN), por **proxies CORS públicos**. O rodapé do mapa mostra a fonte usada; clicar nele abre um diagnóstico com o erro de cada tentativa.
- O Flightradar24 não tem API gratuita oficial: o endpoint usado é não oficial e pode mudar ou recusar o proxy. Para uso comercial, considere uma API licenciada.
- Ao usar um proxy, o serviço do proxy vê a região consultada no mapa.
- A ISS usa `api.wheretheiss.at`.

---

## 🎛 Design do rádio

O botão **🎛 DESIGN** abre o painel de personalização:

- **Estilos**: SR-497HPC (azul), SR-955HP (verde), 148 GTL (preto com display vermelho) e Clássico.
- **Predefinições (estilo PX)**: Cobra 148 GTL, 148 GTL Grafite, Cobra 29 LTD, Galaxy Prata, Uniden Bearcat, President Bronze, Super Star Cromado, Ranger Azul, Connex Verde, Dourado, Vermelho Metal e Gelo Ciano.
- **Acabamentos**: Fosco, Prata, Grafite, Ouro, Bronze, Azul, Vermelho e Verde metalizados, com efeito de metal escovado.
- **Ajuste manual** de cor dos rótulos, do display e do painel, com botão para restaurar.

As predefinições reproduzem apenas combinações de cores inspiradas nesses modelos.

---

## 🤖 API para integração (`window.RA`)

O app expõe um objeto global para integrar assistentes de IA e apps host:

```js
RA.join('PY2ABC', 1, 12, 'Minha sala')   // (indicativo, banda 0-2, canal 1-99, nome)
RA.sendText('Olá, sala!', 'JARVIS')       // publica texto na transcrição
RA.sendAudio(base64, 'audio/mpeg', 'texto falado', 'JARVIS') // transmite áudio na sala
RA.getTranscript()                        // JSON com todo o histórico
RA.getLast(5)                             // últimas N falas
RA.status()                               // sala, indicativo, estações, localização
RA.topics()                               // namespace e tópicos MQTT em uso
RA.onTranscript = e => console.log(e)     // callback a cada nova fala
```

Com o rádio desligado (botão power), `RA.sendAudio` não transmite. Textos enviados por `RA.sendText` também são copiados pela ponte de texto, se ela estiver ligada.

Em apps desktop com **WebView2**, as transcrições são enviadas ao host via `chrome.webview.postMessage` e o host pode enviar `{audio, mime, text, call}` ou `{text, call}` de volta.

---

## 🔒 Privacidade e limitações

- **Áudio**: trafega por WebRTC (criptografado em trânsito), diretamente entre os participantes. Se a conexão direta falhar, passa por um servidor TURN.
- **Salas abertas**: sinalização, presença, transcrições, chat, chamadas seletivas e comandos de moderação passam por **brokers MQTT públicos** e **não são privados**. Qualquer pessoa que conheça o namespace e o tópico da sala pode ler esses metadados.
- **Salas privadas**: a sinalização é autenticada e as transcrições e o chat são cifrados, mas o nome da sala, o canal, a lista de membros (indicativo, nome e localização aproximada) continuam visíveis na presença. Leia os limites em [Salas privadas](#-salas-privadas). Não use para conversas sensíveis.
- **Localização**: só é lida se o navegador permitir e é arredondada para duas casas decimais (~1 km) antes de ser compartilhada com a sala. A mesma posição arredondada é enviada ao **Open-Meteo** para o boletim do canal 9.
- **Tradução**: o texto das falas é enviado ao serviço **MyMemory** para ser traduzido. Deixe **TRADUZIR PARA** desligado se isso for um problema.
- **Dados guardados no navegador** (`localStorage`): indicativo, nome e coordenadas, design do rádio, estado do power, camadas do mapa, favoritos, diário e estatísticas, chave de identidade e as identidades já vistas. Limpar os dados do navegador apaga tudo isso.
- **Gravações**: ficam apenas na memória da aba (as 10 últimas) e somem ao recarregar. Gravar conversas de outras pessoas pode exigir o consentimento delas, conforme a legislação do seu país.
- **Moderação e identidade**: funcionam sem servidor, portanto não impedem um participante modificado de agir fora das regras. A verificação de identidade só prova que a mesma chave está sendo usada de novo.
- **Escalabilidade**: a malha P2P cresce de forma quadrática; o app é pensado para salas pequenas/médias.
- **Disponibilidade**: depende de brokers, relays e APIs públicas de terceiros (MQTT, STUN/TURN, tiles, aviões, ISS, tempo, tradução, proxies CORS), que podem ficar fora do ar ou limitar uso.
- **Transcrição** depende do suporte do navegador à Web Speech API (Chrome/Edge); o idioma está fixado em `pt-BR`.
- **Credenciais TURN**: se for publicar o repositório, **não deixe credenciais reais no código**. Use credenciais próprias, de preferência efêmeras, e rotacione as que já estiverem expostas.

---

## 🌐 Compatibilidade

Navegadores modernos com WebRTC, Web Audio, Web Crypto e `getUserMedia`: Chrome, Edge, Firefox e Safari recentes (a transcrição de fala requer Chrome ou Edge). Funciona em desktop e celular, em modo horizontal e vertical.

Alguns recursos extras dependem de APIs que variam entre navegadores: gravação (`MediaRecorder`), controle por fone (Media Session), notificações, tela sempre ligada, instalação como app e compartilhamento de arquivos do QSL. Onde a API não existe, o recurso simplesmente não funciona e o resto do app segue normal.

---

## 🙏 Créditos e avisos

- Mapas © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors, via [Leaflet](https://leafletjs.com/).
- [MQTT.js](https://github.com/mqttjs/MQTT.js) e [PeerJS](https://peerjs.com/).
- Dados de aeronaves de [Flightradar24](https://www.flightradar24.com/), [adsb.lol](https://adsb.lol/), [adsb.fi](https://adsb.fi/), [adsb.one](https://adsb.one/), [airplanes.live](https://airplanes.live/) e [OpenSky Network](https://opensky-network.org/); posição da ISS por [wheretheiss.at](https://wheretheiss.at/).
- Previsão do tempo por [Open-Meteo](https://open-meteo.com/) e tradução por [MyMemory](https://mymemory.translated.net/).
- Os nomes de modelos citados nos estilos e nas predefinições do painel (Stryker SR-497HPC / SR-955HP, Cobra, Galaxy, Uniden, President, Super Star, Ranger, Connex) pertencem aos seus respectivos fabricantes; o projeto é independente e sem afiliação.

---

## 📄 Licença

Defina a licença do projeto (ex.: MIT) adicionando um arquivo `LICENSE` na raiz do repositório.

---

Feito por **Bruno Nunes da Silva** · DevSoft AI
