# 📻 DevSoft Transceptor

**Rádio PX/CB virtual com voz em tempo real, direto no navegador.**
Um transceptor com visual retrô (painel com knobs, chaves, medidor e display de canal) que conecta pessoas em salas por canal usando WebRTC peer-to-peer, sem servidor próprio e sem build — é um único arquivo `index.html`.

> Autor: **Bruno Nunes da Silva** · DevSoft AI

---

## ✨ Funcionalidades

- **Voz push-to-talk (PTT)** em tempo real entre os participantes da sala, com bloqueio de TX ("travar TX") para toque.
- **99 canais em 3 bandas** (`LOW`, `MID`, `HI`), com frequência exibida no estilo faixa de 27 MHz (ex.: `26.965 MHz` no canal 1 da banda MID).
- **Salas por canal**: crie uma sala no canal sintonizado ou entre em uma sala aberta listada no lobby. Detecção de canal ocupado.
- **Convite por link** (`#sala=M12&n=Nome`) com Web Share API ou cópia para a área de transferência.
- **Modos de recepção**: `CW`, `FM`, `AM`, `USB` e `LSB`, cada um com filtros e nível de ruído próprios.
- **Processamento de áudio realista (Web Audio API)**: squelch, ganho de RF, noise blanker (NB), ruído de fundo, tom de CW, side-tone e beeps de canal ocupado.
- **Mapa das estações** (Leaflet + OpenStreetMap) com a localização aproximada (arredondada a ~1 km) de quem está na sala, se o usuário permitir.
- **Transcrição de fala** via Web Speech API (Chrome/Edge), enviada como texto para a sala, com exportação em **JSON** e **CSV**.
- **Ponte JARVIS / IA**: aceita áudio e texto de um agente externo e transmite na sala; expõe a API global `window.RA` e integra com WebView2 (`chrome.webview`) para apps desktop.
- **Skins do painel** (Stryker SR-497HPC, SR-955HP e clássico) com cores personalizáveis, salvas em `localStorage`.
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
| Fallback de conexão | PeerJS (modo hub/cliente) |
| NAT traversal | STUN (Google, Twilio, Cloudflare) e TURN (Metered) |
| Mapa | Leaflet 1.9 + tiles do OpenStreetMap |
| Reconhecimento de fala | Web Speech API |

Dependências são carregadas por CDN (jsDelivr / cdnjs, com fallback para unpkg). Não há `npm install`, bundler nem backend.

---

## 🚀 Como usar

### Rodando localmente

Microfone e geolocalização exigem **contexto seguro** (`https://` ou `localhost`). Abrir o arquivo direto (`file://`) funciona para testar a interface, mas o link de convite só serve para outras pessoas se o app estiver publicado.

```bash
git clone https://github.com/<seu-usuario>/<seu-repo>.git
cd <seu-repo>

# qualquer servidor estático serve:
python3 -m http.server 8080
# depois abra http://localhost:8080
```

### Publicando

Por ser um arquivo estático, funciona em GitHub Pages, Netlify, Vercel, Cloudflare Pages ou qualquer hospedagem estática com HTTPS.

### Entrando em uma sala

1. Informe seu **indicativo** (mín. 3 caracteres, ex.: `PY2ABC`) e, opcionalmente, seu nome.
2. Gire o seletor de canal / banda ou escolha uma sala aberta no lobby.
3. Permita o acesso ao microfone.
4. Segure o botão **MIC** para transmitir (ou use **TRAVAR TX** no celular).
5. Use **🔗 CONVIDAR** para chamar alguém direto para a sua sala.

### Usando outros brokers MQTT

Passe um ou mais brokers pela query string:

```
https://seu-site/?broker=wss://meu-broker:8084/mqtt&broker=wss://outro-broker/mqtt
```

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

Em apps desktop com **WebView2**, as transcrições são enviadas ao host via `chrome.webview.postMessage` e o host pode enviar `{audio, mime, text, call}` ou `{text, call}` de volta.

---

## 🔒 Privacidade e limitações

- **Áudio**: trafega por WebRTC (criptografado em trânsito), diretamente entre os participantes. Se a conexão direta falhar, passa por um servidor TURN.
- **Sinalização, presença e transcrições**: passam por **brokers MQTT públicos** e **não são privados** — qualquer pessoa que conheça o namespace e o tópico da sala pode ler esses metadados. Não use para conversas sensíveis.
- **Localização**: só é lida se o navegador permitir e é arredondada para duas casas decimais (~1 km) antes de ser compartilhada com a sala.
- **Escalabilidade**: a malha P2P cresce de forma quadrática; o app é pensado para salas pequenas/médias.
- **Disponibilidade**: depende de brokers e relays públicos de terceiros, que podem ficar fora do ar ou limitar uso.
- **Transcrição** depende do suporte do navegador à Web Speech API (Chrome/Edge); o idioma está fixado em `pt-BR`.
- **Credenciais TURN**: se for publicar o repositório, **não deixe credenciais reais no código**. Use credenciais próprias, de preferência efêmeras, e rotacione as que já estiverem expostas.

---

## 🌐 Compatibilidade

Navegadores modernos com WebRTC, Web Audio e `getUserMedia`: Chrome, Edge, Firefox e Safari recentes (a transcrição de fala requer Chrome ou Edge). Funciona em desktop e celular.

---

## 🙏 Créditos e avisos

- Mapas © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors, via [Leaflet](https://leafletjs.com/).
- [MQTT.js](https://github.com/mqttjs/MQTT.js) e [PeerJS](https://peerjs.com/).
- Os nomes de modelos usados nas skins do painel (Stryker SR-497HPC / SR-955HP) pertencem aos seus respectivos fabricantes; o projeto é independente e sem afiliação.

---

## 📄 Licença

Defina a licença do projeto (ex.: MIT) adicionando um arquivo `LICENSE` na raiz do repositório.

---

Feito por **Bruno Nunes da Silva** · DevSoft AI
