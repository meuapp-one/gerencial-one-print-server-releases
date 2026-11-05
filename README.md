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
curl -fsSL https://ps.gerencial.one/instalar.sh | sudo GOPS_VERSAO=0.2.3 sh
```

```powershell
$env:GOPS_VERSAO = '0.2.3'; irm https://ps.gerencial.one/instalar.ps1 | iex
```

## Conectar a estação

O instalador só instala e inicia o serviço: ele **não** faz o pareamento. Sem estação conectada, o serviço fica aguardando, sem imprimir nada. Para conectar a estação ao Gerencial One, rode o comando `conectar` (como administrador):

- **Windows** (PowerShell como administrador):

  ```powershell
  gerencial-one-ps conectar
  ```

- **Linux:**

  ```sh
  sudo gerencial-one-ps conectar
  ```

Sem opção, `conectar` usa o ambiente de **produção**. Para o ambiente de **desenvolvimento** (testes), acrescente `--dev` (`conectar --dev`). O comando grava o ambiente, descarta a credencial anterior (ela só vale no ambiente em que foi criada), reinicia o serviço e mostra no terminal o **código de pareamento** (`XXXX-XXXX`) e o **QR code** até a estação ser aprovada. Termina com `Conectado a <URL> (produção)` ou `(desenvolvimento)`. Ctrl+C só para de acompanhar: o serviço continua pareando, e o código pode ser visto depois com `status`. Com uma impressora USB ligada no Linux, o código também sai impresso no papel, com um QR.

No Gerencial One, um administrador abre **Configuração → Impressão**, informa o código e dá um nome à estação. Em seguida, cadastra as impressoras dessa estação. **Procurar impressoras** lista as encontradas no computador e na rede.

### Trocar de ambiente

Numa máquina já instalada, rode `conectar` de novo com o outro ambiente (`conectar --dev` ou `conectar`/`conectar --prod`). Não precisa reinstalar: o comando troca o ambiente, descarta a credencial e pareia a estação outra vez, que precisa ser aprovada no Gerencial One do novo ambiente.

### Estações instaladas antes do `conectar`

Uma estação já pareada continua funcionando como **produção**, sem rodar nada. Se a estação for revogada no Gerencial One, o serviço passa a aguardar (mesmo depois de reiniciar), e é preciso rodar `conectar`: só ele autoriza um novo pareamento.

## Conexões das impressoras

- **Rede:** o IP da impressora e a porta (normalmente 9100).
- **USB no Windows:** pelo spooler. Instale a impressora no Windows, com o driver do fabricante ou o "Generic / Text Only", e cadastre o nome exato que aparece em Impressoras e scanners.
- **USB no Linux:** o dispositivo `/dev/usb/lp0` (ou `lp1`, …).
- **Bluetooth no Windows:** pareie a impressora nas Configurações do Windows e crie a porta COM de saída (Bluetooth e dispositivos → Mais configurações de Bluetooth → Portas COM → Adicionar → Saída). Cadastre como serial, com a porta criada (ex.: `COM5`).
- **Bluetooth no Linux:** instale com `install --bluetooth` e escolha a impressora em **Procurar impressoras**.

## Referência de comandos

Forma geral: `gerencial-one-ps <comando> [opções]`. No Linux o programa fica em `/usr/local/bin` e, no Windows, o `install` põe `C:\Program Files\GerencialOnePrintServer` no `PATH` do sistema: depois da instalação, **abra um novo terminal** e rode `gerencial-one-ps` de qualquer pasta. Nos exemplos abaixo, `$exe` é o comando (no Windows, em um PowerShell **como administrador**):

```powershell
$exe = "gerencial-one-ps"
```

Estações que já tinham o nome antigo (`gerencial-one-print-server`) passam ao nome novo sozinhas na primeira atualização automática (no Linux o nome antigo continua funcionando como atalho). Em estação migrada pela auto-atualização, o serviço (sessão 0) não consegue avisar o Explorer do usuário: o comando `gerencial-one-ps` fica disponível no terminal depois de sair e entrar na sessão do Windows ou reiniciar a estação (ou de rodar `install` de novo, em um PowerShell de administrador). O serviço grava o `PATH` só nessa migração, uma vez, para não recolocar uma entrada que um administrador tenha removido de propósito.

Opções gerais, valem para o programa e para qualquer comando: `--help` (`-h`) mostra a ajuda e `--version` (`-V`, só no programa) mostra a versão.

| Comando | O que faz | Exige administrador/root | Linux | Windows |
|---|---|---|---|---|
| `run` | Executa o serviço em primeiro plano; é o que o systemd e o Windows Service chamam | não (o serviço roda pelo gerenciador) | sim | sim |
| `conectar` | Conecta a estação ao Gerencial One e mostra o código e o QR de pareamento | sim | sim | sim |
| `install` | Instala e inicia o serviço (não pareia) | sim | sim | sim |
| `uninstall` | Remove o serviço | sim | sim | sim |
| `status` | Mostra o serviço, o ambiente, a estação, o último contato e as impressoras | para ler a credencial e o código de pareamento | sim | sim |
| `logs` | Acompanha os logs (Ctrl+C sai) | no Windows, para ler os logs; no Linux, root ou o grupo `systemd-journal` | sim | sim |
| `scan` | Procura impressoras acessíveis a esta máquina | não (no Linux, `sudo` ajuda a enxergar USB e serial) | sim | sim |
| `doctor` | Diagnostica o ambiente e sugere correções | para ler a configuração | sim | não disponível |

Nota: existe também um comando interno e oculto, `aplicar-atualizacao`, chamado pelo serviço no Linux para aplicar uma atualização; não é de uso manual.

#### `run`

Executa o serviço até receber um sinal (Ctrl+C ou parada do serviço). Sem estação conectada, só registra no log que aguarda o `conectar`. Normalmente você não o chama: o serviço instalado já faz isso.

#### `conectar`

Conecta (ou reconecta) a estação. Exige administrador/root e o serviço instalado.

- `conectar`: ambiente de produção (o padrão).
- `conectar --prod`: o mesmo, explícito.
- `conectar --dev`: ambiente de desenvolvimento (testes).

`--dev` e `--prod` não podem ser usados juntos. Se o executável instalado não tiver a URL de produção, `conectar` avisa e não altera nada; use `--dev`.

```sh
sudo gerencial-one-ps conectar
sudo gerencial-one-ps conectar --dev
```

```powershell
& $exe conectar
& $exe conectar --dev
```

#### `install`

Instala o serviço, cria o usuário e as pastas, e o inicia. Pode ser repetido com segurança: atualiza o executável e mantém a credencial. Não faz o pareamento; rode `conectar` depois.

- `install --bluetooth`: instala o BlueZ (impressoras Bluetooth) sem perguntar. **Só no Linux**; no Windows a opção é ignorada com um aviso, e o Bluetooth é pareado nas Configurações do Windows.

```sh
sudo gerencial-one-ps install
sudo gerencial-one-ps install --bluetooth
```

```powershell
& $exe install
```

#### `uninstall`

Remove o serviço e o executável. A credencial e a configuração só são apagadas com confirmação ou com `--purge`.

- `uninstall --purge`: apaga também a credencial, a configuração e (no Windows) os logs, sem perguntar.

```sh
sudo gerencial-one-ps uninstall
sudo gerencial-one-ps uninstall --purge
```

```powershell
& $exe uninstall
& $exe uninstall --purge
```

#### `status`

Mostra a versão, a situação do serviço, o ambiente (`produção`, `desenvolvimento` ou `personalizado`) e a URL, a estação (ou `aguardando conectar`, ou `pareamento em andamento` com o código), o último contato e as impressoras configuradas. Sem permissão para ler a configuração, mostra o que pode e pede administrador/root.

```sh
sudo gerencial-one-ps status
```

```powershell
& $exe status
```

#### `logs`

Mostra o fim dos logs e continua acompanhando. No Linux segue o journal do systemd, o que exige root (`sudo`) ou pertencer ao grupo `systemd-journal`; no Windows, o arquivo de log mais recente, lido em um terminal de administrador.

```sh
sudo gerencial-one-ps logs
```

```powershell
& $exe logs
```

#### `scan`

Procura impressoras acessíveis a esta máquina (rede, USB, serial e Bluetooth; no Windows, o spooler e as portas COM). Leva cerca de 10 segundos. Não exige administrador, mas no Linux o `sudo` ajuda: sem ele, `/dev/usb/lp*` e as portas seriais podem não ser acessíveis ao seu usuário e deixar de aparecer.

```sh
sudo gerencial-one-ps scan
```

```powershell
& $exe scan
```

#### `doctor`

Diagnóstico: confere o serviço, a credencial, os grupos do usuário de serviço, as permissões dos dispositivos, o BlueZ, o relógio, o acesso ao Gerencial One e conflitos com CUPS e ModemManager, com a correção sugerida para cada problema. **Só no Linux**; no Windows responde que não está disponível (use `status`, `logs` e `scan`).

```sh
sudo gerencial-one-ps doctor
```

#### Opções gerais

```sh
gerencial-one-ps --version
gerencial-one-ps --help
gerencial-one-ps conectar --help
```

```powershell
& $exe --version
& $exe --help
```

Rodar o instalador de novo é seguro: ele atualiza o executável e mantém a credencial e o pareamento.

## Atualização automática

O Print Server se atualiza sozinho quando o Gerencial One libera uma versão nova. Ele baixa o manifesto da versão, confere a **assinatura ed25519** e o **sha256** do executável e só então faz a troca, sem interromper uma impressão em andamento. O serviço reinicia em seguida. Se algo falhar, a versão atual continua funcionando e a atualização é tentada de novo mais tarde.

Cada versão publica `manifesto.json` e `manifesto.json.sig`, além de um `.sha256` ao lado de cada executável. O Print Server só aceita manifestos assinados pela chave do Gerencial One, embutida no próprio executável.
