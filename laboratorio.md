
## 📘 Índice

1. Configuração inicial
   
Passo 1.1: Subir a topologia com os dois nós (node-a e node-b).

  ```
   sudo clab deploy -t <nome da topologia>.yml
  ```
Passo 1.2: Acessar os Nós do Laboratório

   Abra dois terminais diferentes para monitorar os dois nós simultaneamente.
   
   - Para acessar o node-a:
   
   ```
   docker exec -it clab-ebpf-lab-node-a bash
   ```
   
   - Para acessar o node-b:
   
   ```
   docker exec -it clab-ebpf-lab-node-b bash
   ```

Passo 1.3: Verificar Interfaces e Endereços IP

Dentro de cada nó, verifique se a interface eth1 foi criada e se o IP correto foi atribuído pelo script de inicialização.

```
ip addr show dev eth1
```

📝 Pergunta para o Relatório: Qual é o endereço MAC (Link/Ether) associado à interface eth1 no node-a e no node-b?
 
Passo 1.4: Verificar endereços IP configurados com .
   ```
   ip addr show
   ```

Passo 1.5: Testar conectividade básica com ping.
   ```
   ping -c 4 10.0.0.2
   ```

2. Captura de pacotes

   Nesta etapa, começaremos a monitorar e registrar o tráfego da rede a nível de bits e bytes.

   Passo 2.1: Monitoramento em Tempo Real
   No node-a, inicie o tcpdump para escutar na interface que conecta ao node-b:

   ```
   tcpdump -i eth1 -n
   ```
(O parâmetro -n impede a resolução de nomes, exibindo apenas os IPs numéricos).

Com o tcpdump rodando no node-a, vá até o terminal do node-b e envie pings de volta:

```
ping -c 3 10.0.0.1
```
Observe a saída no terminal do node-a. Pressione Ctrl + C para parar o tcpdump. 

Passo 2.2: Salvando Capturas para Análise Externa (.pcap)

Ferramentas de linha de comando são ótimas, mas o Wireshark (GUI) permite análises visuais profundas. Vamos salvar o tráfego em um arquivo padronizado. 

No node-a, execute:

   ```
   tcpdump -i eth1 -n -w /tmp/ping_captura.pcap
   ```

No node-b, gere tráfego gerando pings ou pacotes quaisquer.

Pare a captura no node-a com Ctrl + C.

Para fins didáticos, você pode visualizar o arquivo bruto dentro do próprio container com:

   ```
   tcpdump -r /tmp/ping_captura.pcap -v
   ```
(Nota: O arquivo .pcap gerado na pasta compartilhada ou extraído via docker cp poderá ser aberto no Wireshark da sua máquina física para a etapa 3).

3. Análise de Tráfego e Estrutura de Protocolos

Abra o arquivo .pcap gerado (ou analise detalhadamente a saída do tcpdump -v).

Exercício Prático de Análise:

- Examine um pacote ICMP do tipo Echo Request e um Echo Reply. Responda e documente:

- Camada 2 (Enlace - Ethernet): Identifique o endereço MAC de Origem e de Destino. Eles batem com os coletados no Passo 1.3?

- Camada 3 (Rede - IPv4): * Localize o campo TTL (Time to Live). Qual o valor inicial dele?

- O que acontece com o Checksum se algum bit do cabeçalho for alterado?

- Camada 4 (Mensagem de Controle - ICMP): * Qual o valor do campo Type e Code para o Echo Request?

4. Testes Avançados com Ferramentas Adicionais

A imagem netshoot vem com ferramentas cruciais para sysadmins e engenheiros de redes. Vamos explorar três cenários.

📊 Cenário A: Medição de Largura de Banda com iperf3
O iperf3 atua em modo Cliente-Servidor para testar a capacidade máxima de vazão (Throughput) do link.

Mude o node-b para o modo Servidor:

   ```
   iperf3 -s
   ```
No node-a, execute o Cliente apontando para o servidor:

```
iperf3 -c 10.0.0.2
```
Aguarde 10 segundos e anote a taxa de transferência obtida (Mbits/sec ou Gbits/sec). 

🔌 Cenário B: Simulação de Conexões TCP/UDP com netcat (nc)
O netcat é o "canivete suíço" do TCP/IP. Permite abrir portas e enviar strings de texto puras.

No node-b, abra uma escuta na porta TCP 8080:

   ``` 
    nc -l -p 8080
   ```
No node-a, conecte-se a essa porta:

   ```
   nc 10.0.0.2 8080
   ```
Digite uma mensagem no terminal do node-a (ex: "Olá, Node B! Teste de socket TCP funcionando.") e aperte Enter. Verifique se o texto apareceu no node-b.

🔍 Cenário C: Consultas DNS locais com dig / nslookup
Mesmo sem um servidor DNS configurado nesta sub-rede simples, entender como as ferramentas estruturam pacotes DNS é fundamental.

No node-a, prepare o tcpdump em background ou em outra aba para monitorar a porta 53 (DNS): tcpdump -i any port 53

Tente fazer uma consulta forçando um servidor externo fake (ou um real se seu host tiver saída para internet):

```
dig @8.8.8.8 google.com
```
Caso não haja saída para a internet, avalie a tentativa de envio do pacote UDP na porta 53 usando a captura do tcpdump. 

5. Simulação de Falhas e Resolução de Problemas (Troubleshooting)
   
Um bom técnico se destaca quando a rede falha. Vamos provocar anomalias propositalmente para analisar o comportamento dos protocolos.

Causa de Falha 1: Interface Inativa (Link Down)
No node-a, derrube a interface de comunicação:

```
ip link set eth1 down
```
Execute um ip addr show dev eth1 e observe o estado (state DOWN).

Tente dar um ping 10.0.0.2. Qual erro imediato o sistema operacional exibe?

Restabeleça o link: ip link set eth1 up.

Causa de Falha 2: Máscara de Rede Incorreta (Misconfiguration)
A máscara define os limites da sub-rede. Vamos quebrar essa lógica.

No node-b, mude a máscara da interface mudando o IP para uma sub-rede diferente (ex: /30 que engloba apenas de 10.0.0.1 até 10.0.0.3, ou altere drasticamente para outra rede):

```
ip addr del 10.0.0.2/24 dev eth1
ip addr add 10.0.0.2/30 dev eth1
```
Agora, altere temporariamente o IP do node-a para 10.0.0.5/30:

```
ip addr del 10.0.0.1/24 dev eth1
ip addr add 10.0.0.5/30 dev eth1
```
No node-a, tente pingar 10.0.0.2.

Abra o tcpdump -i eth1 em ambos os nós enquanto pinga.

💡 Desafio: Os pacotes chegam a sair da placa? O protocolo ARP consegue resolver o endereço MAC nesse cenário? Justifique suas observações.

Antes de prosseguir, retorne as configurações originais (10.0.0.1/24 e 10.0.0.2/24) para não comprometer os próximos passos.
