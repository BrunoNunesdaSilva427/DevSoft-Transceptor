# 📻 DevSoft Transceptor

**Rádio PX/CB virtual com voz em tempo real, direto no navegador.**
Um transceptor com visual retrô (painel com knobs, chaves, botão power, medidor e display de canal) que conecta pessoas em salas por canal usando WebRTC peer-to-peer, sem servidor próprio e sem build — é um único arquivo `index.html`.

> Autor: **Bruno Nunes da Silva** · DevSoft AI

---

## ✨ Funcionalidades

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
| Sinalização, presença e transcrição | MQTT 5 sobre WebSocket (`mqtt.js`), com vários brokers públicos em paralelo (EMQX, HiveMQ, Mosquitto, Eclipse, shiftr.io) |
| Salas privadas | Web Crypto API: PBKDF2-SHA-256, ECDH P-256, AES-GCM e HMAC-SHA-256 |
| Fallback de conexão | PeerJS (modo hub/cliente) |
| NAT traversal | STUN (Google, Twilio, Cloudflare) e TURN (Metered) |
| Mapa | Leaflet 1.9 + tiles do OpenStreetMap |
| Aviões e ISS | Flightradar24, adsb.lol, adsb.fi, adsb.one, airplanes.live, OpenSky e wheretheiss.at |
| Reconhecimento de fala | Web Speech API |

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

## 🔐 Salas privadas

Ao criar uma sala (pelo formulário do lobby ou pelo botão **CRIAR SALA AQUI**) você pode definir uma senha. Deixar em branco cria uma sala aberta.

- **Senha**: mínimo de 8 caracteres, digitada em campos mascarados, com confirmação.
- **Validade**: *Permanente* (vale enquanto a sala existir) ou *temporária* (5 min, 15 min, 30 min ou 1 hora). Depois do prazo, **ninguém novo consegue entrar** com aquela senha; quem já está na sala continua. Na lista, a sala aparece como «senha expirada» com o botão **🔒 FECHADA**.
- **Como funciona**: a sala tem uma chave aleatória de 256 bits, e a senha serve apenas como porta de entrada. A presença pública não contém salt nem verificador da senha, só um identificador dessa chave, que não permite adivinhar a senha. Quem quer entrar pede a chave a um membro online por um handshake cifrado com ECDH; o membro só a entrega se a prova da senha (PBKDF2 com 600 mil iterações) for válida. Com a chave, o áudio de sinalização é autenticado e as transcrições são cifradas com AES-GCM.
- **Proteções**: cada membro limita tentativas (3 erros bloqueiam o solicitante por 2 minutos, com teto global de falhas).
- **Sala some quando esvazia**: como a chave só existe na memória dos membros, uma sala privada precisa de pelo menos um membro online para aceitar novas entradas.
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

Com o rádio desligado (botão power), `RA.sendAudio` não transmite.

Em apps desktop com **WebView2**, as transcrições são enviadas ao host via `chrome.webview.postMessage` e o host pode enviar `{audio, mime, text, call}` ou `{text, call}` de volta.

---

## 🔒 Privacidade e limitações

- **Áudio**: trafega por WebRTC (criptografado em trânsito), diretamente entre os participantes. Se a conexão direta falhar, passa por um servidor TURN.
- **Salas abertas**: sinalização, presença e transcrições passam por **brokers MQTT públicos** e **não são privadas**. Qualquer pessoa que conheça o namespace e o tópico da sala pode ler esses metadados.
- **Salas privadas**: a sinalização é autenticada e as transcrições são cifradas, mas o nome da sala, o canal, a lista de membros (indicativo, nome e localização aproximada) continuam visíveis na presença. Leia os limites em [Salas privadas](#-salas-privadas). Não use para conversas sensíveis.
- **Localização**: só é lida se o navegador permitir e é arredondada para duas casas decimais (~1 km) antes de ser compartilhada com a sala.
- **Escalabilidade**: a malha P2P cresce de forma quadrática; o app é pensado para salas pequenas/médias.
- **Disponibilidade**: depende de brokers, relays e APIs públicas de terceiros (MQTT, STUN/TURN, tiles, aviões, ISS, proxies CORS), que podem ficar fora do ar ou limitar uso.
- **Transcrição** depende do suporte do navegador à Web Speech API (Chrome/Edge); o idioma está fixado em `pt-BR`.
- **Credenciais TURN**: se for publicar o repositório, **não deixe credenciais reais no código**. Use credenciais próprias, de preferência efêmeras, e rotacione as que já estiverem expostas.

---

## 🌐 Compatibilidade

Navegadores modernos com WebRTC, Web Audio, Web Crypto e `getUserMedia`: Chrome, Edge, Firefox e Safari recentes (a transcrição de fala requer Chrome ou Edge). Funciona em desktop e celular, em modo horizontal e vertical.

---

## 🙏 Créditos e avisos

- Mapas © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors, via [Leaflet](https://leafletjs.com/).
- [MQTT.js](https://github.com/mqttjs/MQTT.js) e [PeerJS](https://peerjs.com/).
- Dados de aeronaves de [Flightradar24](https://www.flightradar24.com/), [adsb.lol](https://adsb.lol/), [adsb.fi](https://adsb.fi/), [adsb.one](https://adsb.one/), [airplanes.live](https://airplanes.live/) e [OpenSky Network](https://opensky-network.org/); posição da ISS por [wheretheiss.at](https://wheretheiss.at/).
- Os nomes de modelos citados nos estilos e nas predefinições do painel (Stryker SR-497HPC / SR-955HP, Cobra, Galaxy, Uniden, President, Super Star, Ranger, Connex) pertencem aos seus respectivos fabricantes; o projeto é independente e sem afiliação.

---

## 📄 Licença

Defina a licença do projeto (ex.: MIT) adicionando um arquivo `LICENSE` na raiz do repositório.

---

Feito por **Bruno Nunes da Silva** · DevSoft AI
