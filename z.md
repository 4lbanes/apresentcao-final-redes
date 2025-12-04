# Relatorio do Projeto Final - Redes Convergentes

## 1. Por que e o que foi feito
- Trabalho final em dupla para a disciplina de Redes Convergentes, seguindo o enunciado do PDF `trabalho final 2025-2 redes.pdf`.
- Montamos uma rede com roteamento dinamico (RIP, OSPF e BGP) e dois servicos principais: DHCP para entregar IP automaticamente e VoIP usando Cisco CME para registrar ramais.
- O foco foi mostrar integracao entre protocolos diferentes, troca de rotas entre tres dominios autonomos (AS) e conectividade fim a fim entre computadores em redes distintas.

## 2. Visao geral da rede
- Sao tres blocos de rede (AS) interligados: um com RIP, outro com OSPF em varias areas e um par de roteadores BGP fazendo a ponte entre eles.
- No AS de RIP ficam duas LANs internas; no AS de OSPF ha duas LANs internas e dois enlaces de area; no meio estao os roteadores BGP que conectam tudo.
- Um roteador de voz (CME) fica na mesma LAN do R1 para entregar ramais e option 150.

![Topologia Geral](<Topologia Geral.png>)  
*Desenho completo da rede com os tres AS, links ponto a ponto, LANs e posicao dos roteadores de voz e borda.*

## 3. Quem faz o que na topologia
### AS 65001 - RIP (rede interna)
- **R1**: porta de entrada da LAN 192.168.1.0/24 e ligação com o roteador BGP R6. Anuncia a LAN de dados/voz e o enlace para R2.  
  Prints: ![R1 Config](R1-Config.png) — mostra interfaces, RIP e BGP anunciando a LAN de 192.168.1.0/24 e links para R2 e R6.
- **R2**: porta da LAN 192.168.2.0/24 e ligação com o roteador BGP R7. Anuncia sua LAN e o enlace interno para R1.  
  Prints: ![R2 Config](R2-Config.png) — traz interfaces de LAN e links seriais, mais a configuracao de RIP e BGP anunciando as redes internas.

### AS 65002 - OSPF (rede interna com areas)
- **R3 (Area 0)**: nucleo OSPF; liga as duas areas internas e o roteador BGP R6.  
  Prints: ![R3 Config](R3-Config.png) — interfaces das areas 0, 1 e 2, OSPF ativo e BGP anunciando as redes aprendidas.
- **R4 (Area 1)**: conecta a area 1 ao roteador BGP R7.  
  Prints: ![R4 Config](R4-Config.png) — evidencia o enlace da area 1 e o peering BGP com R7.
- **R5 (Area 2)**: atende a LAN 192.168.5.0/24 na area 2 e sobe para R3.  
  Prints: ![R5 Config](R5-Config.png) — mostra a interface da LAN 192.168.5.0/24 e o anuncio no OSPF.

### AS 65003 / 65004 - BGP (bordas)
- **R6 (AS 65003)**: recebe rotas do AS de RIP (R1) e do AS de OSPF (R3) e faz a troca entre eles; tem um enlace para R7.  
  Prints: ![R6 Config](R6-Config.png) — configuracao BGP com vizinhos de AS 65001 e 65002 e redes dos enlaces publicados.
- **R7 (AS 65004)**: recebe rotas de R2 (RIP) e de R4 (OSPF) e compartilha com R6, fechando o caminho entre os blocos.  
  Prints: ![R7 Config](R7-Config.png) — peering BGP com R2, R4 e R6, com anuncios dos tres enlaces seriais/ethernet.

### Roteador de Voz (CME)
- **R-VoIP**: roteador de voz na LAN 192.168.1.0/24; entrega DHCP com option 150 e registra quatro ramais (2000-2003).  
  Obs.: integracao via RIP com R1 para que a rede de voz seja conhecida na malha.

## 4. Como a rede conversa
- **RIP (AS 65001)**: usado dentro do bloco com R1 e R2 para anunciar as duas LANs e o enlace interno; R1 exporta essas rotas para o BGP.
- **OSPF (AS 65002)**: dividido em area 0 (R3), area 1 (R4) e area 2 (R5); R3 e R4 exportam as rotas OSPF para o BGP.
- **BGP (entre AS)**: R6 e R7 trocam rotas com os AS de RIP e OSPF; cada um publica os enlaces que conecta e repassa as rotas aprendidas.  
  Prints de vizinhanca:  
  ![Vizinhança BGP R1](<Vizinhança-BGP-R1.png>) — sessao BGP de R1 com o AS de borda.  
  ![Vizinhança BGP R2](<Vizinhança-BGP-R2.png>) — sessao BGP de R2 com o AS de borda.  
  ![Vizinhança BGP R3](<Vizinhança-BGP-R3.png>) — sessao BGP de R3 (OSPF) com o AS de borda.  
  ![Vizinhança BGP R4](<Vizinhança-BGP-R4.png>) — sessao BGP de R4 (OSPF area 1) com o AS de borda.
- **Pontos de cuidado**: manter so um DHCP ativo na LAN 192.168.1.0/24 para evitar conflito e, se quiser redundancia extra, pode-se ativar a sessao BGP direta entre R6 e R7 pelo enlace 10.3.5.0/30.

## 5. Servicos de rede
- **DHCP**: R1 e R-VoIP oferecem pool na LAN 192.168.1.0/24 com option 150; recomenda-se deixar apenas o servidor do R-VoIP ativo para simplificar.
- **VoIP (CME)**: quatro ramais prontos (2000-2003) e um telefone IP ja cadastrado; o comando `show ephone` confirma o registro.  
  Print: ![Show ephone](<show ephone.png>) — lista os telefones IP vistos pelo CME e os ramais associados.

## 6. Testes e evidencias
- Pings entre PCs de LANs diferentes comprovam que os tres protocolos estao trocando rotas corretamente.  
  ![Teste PC1 para PC5](<Teste-PC1 para PC5.png>) — PC1 (RIP) alcançando PC5 (OSPF).  
  ![Teste PC2 para PC7](<Teste-PC2 para PC7.png>) — PC2 (RIP) alcançando PC7 (BGP/OSPF).  
  ![Teste PC4 para PC8](<Teste-PC4 para PC8.png>) — PC4 (OSPF) alcançando PC8 (BGP/OSPF).
- As tabelas de roteamento e o `show ip protocols` mostram aprendizado dinamico ativo.  
  ![Show IP Protocols R1](<show ip protocols R1.png>) — confirma RIP ativo no R1.  
  ![R1 Tabela de Roteamento IPv4](<R1-tabela de roteamento IPv4.png>) — rotas aprendidas em R1 (RIP/BGP).  
  ![R3 Tabela de Roteamento IPv4](<R3-tabela de roteamento IPv4.png>) — rotas aprendidas em R3 (OSPF/BGP).

## 7. Conclusao
- A rede entrega conectividade fim a fim integrando RIP, OSPF e BGP como solicitado.
- DHCP e VoIP estao configurados e funcionais, com option 150 e ramais do CME disponiveis.
- As vizinhancas BGP estao ativas e os testes de ping entre LANs confirmam comunicacao plena.
