# PPPwn WRT - Correções Aplicadas

Este fork contém correções para o script de instalação do PPPwn_WRT para funcionar corretamente com o formato atual do OpenWrt.

## Problemas Corrigidos

### 1. Parsing de Portas LAN
**Problema**: O script original não conseguia extrair corretamente as portas LAN da configuração do OpenWrt, que usa `option ports` em vez de `list ports`.

**Correção**: Modificado o comando awk para processar adequadamente a linha de portas múltiplas:
```bash
# Antes (não funcionava)
if ($1 == "list" && $2 == "ports") {
    print $3
}

# Depois (funciona)
if ($1 == "option" && $2 == "ports") {
    for (i = 3; i <= NF; i++) {
        gsub(/[\'\"]/,"", $i)
        if ($i != "") print $i
    }
}
```

### 2. Modificação da Configuração de Rede
**Problema**: O script para modificar a configuração de rede não funcionava com o formato atual do OpenWrt.

**Correção**: Reescrito o processamento para lidar com `option ports` em vez de `list ports`:
```bash
if ($1 == "option" && $2 == "ports") {
    # Process ports line - remove selected ports
    new_ports = ""
    for (i = 3; i <= NF; i++) {
        port = $i
        gsub(/[\'\"]/,"", port)
        # Check if port should be removed
        # ... logic to rebuild ports line
    }
    if (new_ports != "") {
        print "\toption ports " new_ports
    }
}
```

### 3. Caminho do PPPwn
**Problema**: O path estava sendo determinado dinamicamente de forma incorreta.

**Correção**: Definido path fixo correto:
```bash
# Antes
ppwnpath="$(cd "$(dirname "$0")" && pwd)"

# Depois
ppwnpath="/root/PPPwn_WRT-main"
```

## Testado com:
- OpenWrt 24.10.0
- Roteador ASUS RT-AC68U
- Firmware PS4 11.00

## Como usar:
1. Baixe o script corrigido: `wget https://raw.githubusercontent.com/fidelmarques/PPPwn_WRT/fix-network-config/install.sh`
2. Execute: `chmod +x install.sh && ./install.sh`
3. Siga as instruções na tela

## Configuração resultante:
- Interface PS4: `ps4` (bridge com porta LAN selecionada)
- Interface PPPoE: `pppwn` (usando dispositivo `ps4`)
- Credenciais padrão: usuário `ppp`, senha `ppp`
- IP PS4: `192.168.3.11`
- Gateway: `192.168.3.1`
