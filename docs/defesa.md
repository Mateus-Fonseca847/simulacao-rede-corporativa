# Roteiro para a defesa presencial

1. **Visão geral:** mostrar a topologia completa, andar por andar.
2. **Endereçamento:** explicar as VLANs e sub-redes, e por que cada uma existe.
3. **Os 4 requisitos obrigatórios**, um a um: Ethernet+VoIP (1º/2º), Wi-Fi funcionários (todos), Wi-Fi público (térreo), IoT (copas).
4. **O térreo tombado:** explicar a solução adotada para não alterar a estrutura física.
5. **A redução para 10%:** explicar que o arquivo .pkt simula ~30 estações, mas a rede (sub-redes, portas, capacidade de AP) foi dimensionada para 300.
6. **Testar ao vivo:** um `ping`/`traceroute` entre VLANs permitidas, e mostrar que o isolamento (público × corporativo, IoT × dados) funciona.
