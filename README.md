# Kernel
Atividade laboratorial de kernel


# Relatório de Compilação do Kernel Linux

Este documento registra o processo e os resultados da compilação personalizada do kernel Linux, incluindo respostas às questões de reflexão solicitadas.

---

## 1. Processo de Execução

### 1.1. Download e Extração
* **Versão:** 6.6.30 (LTS)
* **Link:** `https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.30.tar.xz`
* **Extração:** `tar -xf linux-6.6.30.tar.xz`

### 1.2. Instalação de Dependências
* **Comando:** `sudo apt install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison`
* **Resultado:** Todos os pacotes foram instalados sem erros.

### 1.3. Configuração
* **Base:** A configuração do kernel em execução (`/boot/config-$(uname -r)`) foi copiada para `.config`.
* **Ferramenta:** `make menuconfig`
* **Modificação Realizada:**
    * **Opção:** `Device Drivers -> Network device support -> Wireless LAN (CONFIG_WLAN)`
    * **Ação:** Desabilitada (`[N]`).
    * **Justificativa:** A máquina virtual de destino não possui hardware Wi-Fi. Esta alteração reduz o tempo de compilação, o tamanho final dos módulos e a superfície de ataque do kernel.

### 1.4. Compilação e Instalação
* **Compilação:** `make -j$(nproc)`
* **Instalação:** `sudo make modules_install` e `sudo make install`
* **Atualização do Bootloader:** `sudo update-grub`
* **Tempo:** Aproximadamente 1h 45min (VM com 2 vCPUs).
* **Observações:** Inúmeras advertências (warnings) do compilador foram exibidas, o que é esperado. Nenhum erro fatal ocorreu.

### 1.5. Verificação
* O sistema foi reiniciado.
* **Comando:** `uname -mrs`
* **Saída:** `Linux 6.6.30 ...`
* **Resultado:** O sistema inicializou com sucesso usando o kernel compilado.

---

## 2. Questões de Reflexão

**1. Qual foi a versão do kernel que você compilou, e por que escolheu essa versão?**
Compilei a versão **6.6.30**. Escolhi esta versão por ser um kernel **LTS (Long-Term Support)**, priorizando estabilidade e suporte de longo prazo em vez de recursos experimentais de uma versão "mainline".

**2. Que configuração você modificou no `menuconfig`? Por que fez essa modificação? O que espera que essa modificação altere no comportamento do sistema?**
Desabilitei o suporte a **Wireless LAN (CONFIG_WLAN)**.
* **Por quê:** A máquina virtual de destino não possui hardware Wi-Fi, tornando os drivers desnecessários.
* **Resultado esperado:** Redução do tempo de compilação, menor consumo de espaço em disco pelos módulos e uma superfície de ataque de segurança reduzida. Não há impacto esperado no comportamento funcional (rede Ethernet).

**3. Durante a compilação você encontrou algum erro, advertência ou comportamento inesperado? Como resolveu ou o que faria para resolver?**
* **Advertências (Warnings):** Centenas de advertências de compilação foram exibidas. Elas são normais em um projeto deste tamanho e foram ignoradas, pois não são erros fatais.
* **Tempo:** O tempo de compilação (1h 45min) foi longo devido aos 2 vCPUs. Para resolver, usei `make -j$(nproc)` para paralelizar a tarefa.

**4. Quais os riscos e benefícios de “rodar” um kernel compilado por você mesmo em comparação com usar o kernel padrão da distribuição?**
* **Benefícios:** Controle granular, otimização (remoção de "bloat"), kernel menor, e habilidade de habilitar recursos experimentais ou patches específicos.
* **Riscos:** Perda de estabilidade (se mal configurado), falhas de boot (se drivers essenciais forem desabilitados) e a transferência da responsabilidade de manutenção e patching de segurança do time da distribuição para o usuário.

**5. Em quais cenários você considera “vale a pena” compilar seu próprio kernel? Cite ao menos dois.**
1.  **Sistemas Embarcados/IoT:** Onde os recursos (RAM, armazenamento) são extremamente limitados e um kernel genérico é inviável.
2.  **Sistemas de Baixa Latência (Real-Time):** Aplicações que exigem patches `PREEMPT_RT` e ajustes finos no scheduler que não estão presentes no kernel padrão.

**6. Se o sistema não inicializar após a compilação/instalação do kernel, qual seria seu plano de contingência para recuperar o sistema?**
O `sudo update-grub` adiciona o novo kernel sem remover o antigo. O plano seria:
1.  Reiniciar a máquina.
2.  No menu do GRUB, selecionar "Opções Avançadas".
3.  Selecionar o kernel anterior (o kernel da distribuição) para inicializar.
4.  Após o boot, analisar o `.config` para corrigir o erro ou simplesmente remover o kernel com falha.
