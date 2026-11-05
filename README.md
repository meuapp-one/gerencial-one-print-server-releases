# Gerencial One Print Server

O Print Server imprime os pedidos do Gerencial One em impressoras térmicas (ESC/POS). Ele é um serviço sem tela: roda num computador da loja (um mini PC com Windows, uma máquina Linux ou um Raspberry Pi), conversa com o Gerencial One pela internet e envia as comandas às impressoras de rede, USB, serial ou Bluetooth.

Este repositório só guarda as versões publicadas: os executáveis, os instaladores e o manifesto assinado da atualização automática. Cada versão fica em [Releases](https://github.com/meuapp-one/gerencial-one-print-server-releases/releases).

## Instalação

### Windows (x64)

Abra o **PowerShell como administrador** (menu Iniciar, digite "PowerShell", clique com o botão direito e escolha "Executar como administrador") e rode:

```powershell
irm https://ps.gerencial.one/instalar.ps1 | iex
```

O instalador baixa o executável, confere o sha256 e registra o serviço **Gerencial One Print Server**, com início automático e reinício em caso de falha. O executável fica em `C:\Program Files\GerencialOnePrintServer\`, e a configuração, a credencial e os logs em `C:\ProgramData\GerencialOnePrintServer\`, acessíveis só a administradores.

### Linux e Raspberry Pi

Em um terminal, rode:

```sh
curl -fsSL https://ps.gerencial.one/instalar.sh | sudo sh
```

Para usar uma impressora **Bluetooth**, instale também o BlueZ:

```sh
curl -fsSL https://ps.gerencial.one/instalar.sh | sudo sh -s -- --bluetooth
```

Funciona em x86_64, aarch64 (Raspberry Pi 3, 4 e 5 com sistema de 64 bits) e armv7 (Raspberry Pi 2, 3 e 4 com sistema de 32 bits). O armv6 (Raspberry Pi Zero e Pi 1) não é suportado. É preciso ter o systemd, que já vem no Raspberry Pi OS, no Debian e no Ubuntu.

O instalador baixa o binário do seu processador, confere o sha256 e instala o serviço `gerencial-one-print-server`, que inicia sozinho no boot. A configuração, a credencial e os logs ficam em `/etc/gerencial-one-print-server/`.

### Uma versão específica

Para instalar uma versão diferente da mais recente, informe-a antes do comando:

```sh
curl -fsSL https://ps.gerencial.one/instalar.sh | sudo GOPS_VERSAO=0.2.1 sh
```

```powershell
$env:GOPS_VERSAO = '0.2.1'; irm https://ps.gerencial.one/instalar.ps1 | iex
```

## Pareamento

Depois da instalação, o Print Server mostra um código de pareamento (`XXXX-XXXX`):

- **Windows:** rode, como administrador, `& "C:\Program Files\GerencialOnePrintServer\gerencial-one-print-server.exe" status`.
- **Linux:** rode `sudo gerencial-one-print-server status` ou veja os logs com `gerencial-one-print-server logs`. Com uma impressora USB ligada, o código também sai impresso no papel, com um QR.

No Gerencial One, um administrador abre **Configuração → Impressão**, informa o código e dá um nome à estação. Em seguida, cadastra as impressoras dessa estação. **Procurar impressoras** lista as encontradas no computador e na rede.

## Conexões das impressoras

- **Rede:** o IP da impressora e a porta (normalmente 9100).
- **USB no Windows:** pelo spooler. Instale a impressora no Windows, com o driver do fabricante ou o "Generic / Text Only", e cadastre o nome exato que aparece em Impressoras e scanners.
- **USB no Linux:** o dispositivo `/dev/usb/lp0` (ou `lp1`, …).
- **Bluetooth no Windows:** pareie a impressora nas Configurações do Windows e crie a porta COM de saída (Bluetooth e dispositivos → Mais configurações de Bluetooth → Portas COM → Adicionar → Saída). Cadastre como serial, com a porta criada (ex.: `COM5`).
- **Bluetooth no Linux:** instale com `--bluetooth` e escolha a impressora em **Procurar impressoras**.

## Comandos úteis

| O quê | Windows (PowerShell como administrador) | Linux |
|---|---|---|
| Situação do serviço, da estação e das impressoras | `& "C:\Program Files\GerencialOnePrintServer\gerencial-one-print-server.exe" status` | `sudo gerencial-one-print-server status` |
| Acompanhar os logs | `… gerencial-one-print-server.exe logs` | `gerencial-one-print-server logs` |
| Procurar impressoras | `… gerencial-one-print-server.exe scan` | `sudo gerencial-one-print-server scan` |
| Diagnóstico | — | `sudo gerencial-one-print-server doctor` |
| Desinstalar (mantém a credencial) | `… gerencial-one-print-server.exe uninstall` | `sudo gerencial-one-print-server uninstall` |
| Desinstalar e apagar tudo | `… gerencial-one-print-server.exe uninstall --purge` | `sudo gerencial-one-print-server uninstall --purge` |

Rodar o instalador de novo é seguro: ele atualiza o executável e mantém a credencial e o pareamento.

## Atualização automática

O Print Server se atualiza sozinho quando o Gerencial One libera uma versão nova. Ele baixa o manifesto da versão, confere a **assinatura ed25519** e o **sha256** do executável e só então faz a troca, sem interromper uma impressão em andamento. O serviço reinicia em seguida. Se algo falhar, a versão atual continua funcionando e a atualização é tentada de novo mais tarde.

Cada versão publica `manifesto.json` e `manifesto.json.sig`, além de um `.sha256` ao lado de cada executável. O Print Server só aceita manifestos assinados pela chave do Gerencial One, embutida no próprio executável.
