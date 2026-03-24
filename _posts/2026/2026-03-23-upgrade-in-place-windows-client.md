---
#layout: post
title: "Atualizando uma VM do Azure com Windows 10 para Windows 11 sem formatação"
date: 2026-03-16 17:30:00 -03:00
categories: [Azure][Windows]
tags: [azure, windows-client, windows-10, windows-11, upgrade]
slug: 'upgrade-in-place-windows-client-azure-vm'

image:
  path: assets/img/006/001-windows-client-upgrade-azure.png
---

Fala PessoALL! Tudo certo?

Quando falamos de atualização de **Windows Client** dentro do Azure, só pensamos em uma única saída: **Subir uma nova VM do zero já com a versão mais nova do sistema operacional**

E sendo bem sincero? Em muitos cenários, criar uma nova VM ainda continua sendo o caminho mais limpo, porém essa não é a maneira mais prática.

Imagine aquele ambiente onde você já tem uma VM com Windows 10 pronta, com aplicativos instalados, atalhos configurados, perfil de usuário ajustado, acesso remoto funcionando, políticas aplicadas e uma rotina operacional já bem amarrada. Em cenários assim, reconstruir tudo do zero pode gerar retrabalho desnecessário.

É justamente aqui que o **upgrade in-place** pode fazer sentido.

No artigo anterior onde falamos sobre como [Atualizamos o Windows Server em uma VM do Azure sem surpresas](https://blog.ruizsolutions.online/posts/upgrade-in-place-windows-server-azure-vm/), agora falaremos como realizar o mesmo procedimento, porém em uma VM contendo um Windows 10

**Neste artigo, realizaremos um *upgrade in-place* de uma VM Windows Client no Azure, entendendo os pré-requisitos, os cuidados necessários, os limites desse processo e a execução da atualização de forma organizada, prática e suportada.**

---

## Quando esse método faz sentido?

Antes de sair clicando em tudo, precisamos alinhar as expectativas.

O upgrade in-place de Windows Client costuma fazer bastante sentido quando você tem:

- Uma VM já em produção com aplicações configuradas;
- Perfis, acessos e customizações que dariam trabalho para recriar;
- Necessidade de atualizar o sistema operacional sem mexer demais na estrutura atual.

E aqui vai um ponto importante: a própria Microsoft recomenda que, ao sair de **Windows 10 para Windows 11**, a melhor prática seja **implantar novas VMs**, justamente para evitar problemas de compatibilidade e garantir uma configuração mais otimizada.

> Isso não significa que o upgrade in-place não possa funcionar. Significa apenas que você precisa saber exatamente onde ele faz sentido e onde ele pode te dar mais trabalho do que benefício. 
{: .prompt-warning }

---

### Pre-requisitos

Antes de começar, valide os seguintes pontos:

- Possuir permissão de no mínimo **Contributor** da Subscription;
- Validar previamente se a VM é **Windows 10 single-session**;
- Confirmar que a VM **não utiliza [OS Disk efêmero](https://learn.microsoft.com/pt-br/azure/virtual-machines/ephemeral-os-disks)**, pois esse cenário não é suportado;
- Criar **snapshot** do disco do sistema operacional e, se houver disco de dados também;
- Validar se a VM atende aos requisitos do sistema operacional de destino, para upgrade para Windows 11, validar especialmente:
  - **4 GB ou mais de memória RAM**;
  - **64 GB ou mais de armazenamento**;
  - **UEFI / Secure Boot capable**;
  - **TPM 2.0 / vTPM**;
- Validar se a VM não faz parte de um cenário **Azure Virtual Desktop pooled host pool**, pois esse tipo de in-place upgrade não é suportado;
- *(opcional)* Executar a ferramenta **Azure VM Windows OS Upgrade Assessment Tool** antes da mudança.

---

## Mão na massa!

#### Passo 1

A primeira etapa é confirmar se a VM realmente está dentro de um cenário suportado.

Hoje, a documentação da Microsoft informa suporte para **Windows 10 single-session, todas as edições e versões**, dentro do contexto de atualização do sistema em VMs Windows no Azure.

Mas aqui existe um detalhe importantíssimo:

- **Single-session:** suportado;
- **Single-session para multi-session:** não suportado;
- **Session host em pooled host pool no AVD:** não suportado para in-place upgrade;

Ou seja: antes de pensar no “como”, primeiro precisamos validar o “onde”.

![winclient-upgrade](assets/img/006/002-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

---

A Microsoft sugere que executemos **[Ferramenta de avaliação de atualização do sistema operacional Windows da VM do Azure](https://learn.microsoft.com/pt-br/troubleshoot/azure/virtual-machines/windows/windows-vm-osupgradeassessment-tool)** onde valida se o Sistema Operacional possui compatibilidade com o modelo de upgrade in-place. É uma ferramenta bem simples e de fácil utilização, onde você irá realizar o download da ferramenta diretamente do [repositório oficial no Github](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/Windows_OSUpgrade_Assessment_Validation) para a VM que será realizada essa atualização e executá-lo.

Caso ocorra alguma falha, ele retornará uma imagem parecida com essa:
![winclient-upgrade](assets/img/006/003-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

> Esse é o tipo de validação que te salva antes da janela de manutenção começar. 
{: .prompt-tip }

---

#### Passo 2

Agora vamos executaremos a **[Ferramenta de avaliação de atualização do sistema operacional Windows da VM do Azure](https://learn.microsoft.com/pt-br/troubleshoot/azure/virtual-machines/windows/windows-vm-osupgradeassessment-tool)**. Essa ferramenta ajuda a validar se a VM está pronta para o processo e também identifica alguns impedimentos comuns antes da atualização. Ela nos auxiliará a verificar os pontos abaixo:

- Versão atual do sistema operacional;
- Caminho de upgrade suportado;
- Espaço em disco;
- Memória RAM;
- Recursos de segurança da VM, como:
  - Trusted Launch;
  - Secure Boot;
  - vTPM.

1 - Baixe o [Azure VM Windows OS Upgrade Assessment Tool](https://github.com/Azure/azure-support-scripts/tree/master/RunCommand/Windows/Windows_OSUpgrade_Assessment_Validation) diretamente do repositório oficial do GitHub na VM que será atualizada;
![winclient-upgrade](assets/img/006/004-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

![winclient-upgrade](assets/img/006/005-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

2 - Agora abra um PowerShell com **privilégios elevados de administrador**, navegue até onde foi salvo o Windows_OSUpgrade_Assessment_Validation.ps1 e execute-o;
![winclient-upgrade](assets/img/006/006-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

3 - Analise o resultado antes de seguir para as próximas etapas. Caso o resultado for negativo, precisaremos analisar os requisitos, ajusta-los e só então poderemos prosseguir.
![winclient-upgrade](assets/img/006/007-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

> Como podemos notar, tivemos 4 pontos de falha: Trusted Launch, Secure Boot, Virtual TPM e Physical Memory (4GB)
{: .prompt-warning }

Vamos navegar no portal do Azure e identificar o que houve e como podemos ajustar:
![winclient-upgrade](assets/img/006/008-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

> Localizados os verdadeiros ofensores, podemos de fato trabalhar em ajustar para refletir aos **pré-requisitos** cidados acima
{: .prompt-info }

4 - Primeiramente precisaremos desligar/desalocar a VM, caso contrário, nenhuma atividade será permitida:
![winclient-upgrade](assets/img/006/009-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

5 - Para habilitarmos o Trusted Launch, Secure Boot e Virtual TPM, basta navegarmos até Configurações > Security Type e mudarmos de *Standard* para **Trusted launch virtual machines**:
![winclient-upgrade](assets/img/006/010-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

![winclient-upgrade](assets/img/006/011-windows-client-upgrade-azure.png){: .shadow .rounded-10 }
<br>

> Como podemos notar, tivemos 4 pontos de falha: Trusted Launch, Secure Boot, Virtual TPM e Physical Memory (4GB)
{: .prompt-warning }

---

#### Passo 3

Agora vamos revisar os requisitos mínimos e o cenário da VM.

Para upgrade de Windows Client no Azure, a documentação orienta que a VM tenha pelo menos:

- **2 GB de RAM**;
- **12 GB livres no disco do sistema**.

Mas quando falamos especificamente de **Windows 11**, vale trabalhar com o que a própria Microsoft exige para o sistema operacional:

- **4 GB de RAM**;
- **64 GB de armazenamento**;
- **UEFI**;
- **Secure Boot**;
- **TPM 2.0**.

Então aqui vai minha recomendação prática: se o alvo é Windows 11, não trabalhe no “mínimo do mínimo”. Trabalhe no que o Windows 11 realmente espera encontrar.

Além disso:

1 - Confirme se a VM **não está usando OS disk efêmero**;

2 - Revise se a geração da VM e os recursos de segurança estão coerentes com o destino;

3 - Valide se você não está tentando atualizar um cenário **multi-session / pooled host pool** via in-place, porque aí o caminho é outro.

> Esse é um daqueles momentos onde cinco minutos de validação podem economizar horas de troubleshooting depois. 
{: .prompt-warning }

---

#### Passo 4

Como todo administrador, você precisa pensar sempre em rollback.

Antes da atualização, faça backup da VM. A própria documentação da Microsoft recomenda o uso do **Azure Backup**, embora você também possa utilizar outra solução de backup confiável.

Mais do que isso: o ideal é **validar a restauração**. Não adianta apenas dizer “o backup foi criado”. O que importa é saber se ele realmente te devolve uma VM funcional caso algo dê errado no meio do caminho.

1 - Configure ou valide o backup da VM;

2 - Execute o backup antes da janela de mudança;

3 - Sempre que possível, confirme que uma restauração seria viável.

> Upgrade in-place sem backup é aposta. E ambiente corporativo não combina com aposta. 
{: .prompt-danger }

---

#### Passo 5

Com a VM validada e protegida, podemos partir para o processo em si.

Diferente do Windows Server, aqui não existe aquela etapa de criar um disco especial de upgrade via Marketplace. Para Windows Client compatível, o fluxo suportado passa pelo próprio **Windows Update**.

1 - Conecte-se à VM via **RDP** ou **Azure Bastion**;

2 - Acesse:

**Configurações > Atualização e Segurança > Windows Update**

3 - Clique em **Verificar se há atualizações**;

4 - Aguarde até que a **Feature Update** correspondente apareça;

5 - Quando a atualização for exibida, clique em **Baixar e instalar agora**.

> Se a Feature Update não aparecer, pare e revise o assessment, os requisitos do Windows 11, o modelo da VM e os recursos de segurança. Não tente “forçar” no improviso antes de validar o motivo. 
{: .prompt-warning }

---

#### Passo 6

Depois que a atualização começar, o processo fica bem parecido com o que já conhecemos em máquinas físicas ou VMs locais.

A atualização será baixada, aplicada e a VM será reiniciada automaticamente. Durante esse período, a sua sessão remota será desconectada, e isso é totalmente esperado.

Nesse momento, você pode acompanhar o andamento por meio de:

- **RDP**, quando a máquina voltar;
- **Azure Bastion**;
- **Boot Diagnostics / Screenshot** no portal do Azure.

Isso ajuda bastante a entender se a máquina ainda está no meio da atualização ou se já voltou para um estado operacional.


> Lembre-se: o screenshot do Boot Diagnostics não se atualiza sozinho. Use o botão de **Refresh** para acompanhar a evolução do processo. 
{: .prompt-info }

---

#### Passo 7

Concluído o upgrade, chegou a hora da parte mais importante: validar se a operação voltou de forma saudável.

Aqui você deve conferir pelo menos os seguintes pontos:

- Versão atual do Windows;
- Aplicações instaladas;
- Acesso remoto;
- Conectividade de rede;
- Perfil do usuário;
- Ativação/licenciamento;
- Performance geral;
- Integridade do ambiente após a atualização.

Se o upgrade foi de Windows 10 para Windows 11, valide também se todos os recursos de segurança e compatibilidade continuam funcionando como esperado.


> Não trate “a VM ligou” como sinônimo de “a mudança acabou”. O que precisa voltar é a experiência do usuário e a operação. 
{: .prompt-warning }

---

#### Passo 8

Depois da validação, faça a limpeza e o fechamento da mudança.

1 - Execute a limpeza de disco;

2 - Remova instalações anteriores e arquivos temporários, quando aplicável;

3 - Reative temporariamente qualquer proteção que tenha sido desabilitada antes da atualização;

4 - Mantenha o backup por uma janela segura;

5 - Documente a mudança realizada.

Essa limpeza ajuda bastante a recuperar espaço e evitar carregar arquivos desnecessários do processo de atualização.


> Utilize a limpeza de disco e remova instalações anteriores quando tiver certeza de que a VM está estável e o rollback não será mais necessário. 
{: .prompt-warning }

---

#### Passo 9

Agora vem o alerta mais importante do artigo.

Assim como acontece com Windows Server, quando você realiza um upgrade in-place em uma VM Windows no Azure, isso causa uma desconexão entre o **plano de dados** e o **plano de controle** da VM.

Na prática, isso significa que recursos do Azure como:

- **Automatic guest patching**;
- **Automatic OS image upgrades**;
- **Hotpatching**;
- **Azure Update Manager**

não ficam disponíveis para essa VM após esse tipo de processo.

Além disso, as propriedades da imagem no portal do Azure, como **publisher**, **offer** e **plan**, podem continuar refletindo a imagem original, mesmo que o sistema operacional dentro da VM tenha sido atualizado.


> Em outras palavras: o sistema operacional foi atualizado, mas a VM não virou uma “nova imagem nativa” do Azure. 
{: .prompt-warning }

---

#### Passo 10

E se não der suporte? Ou se a VM simplesmente não estiver pronta para esse modelo?

A própria Microsoft documenta uma alternativa mais trabalhosa, mas válida em cenários sem suporte direto: baixar o **VHD**, anexá-lo a uma VM local no **Hyper-V**, executar o upgrade localmente e depois enviar esse VHD novamente para o Azure.

É um caminho possível, mas sinceramente? Eu trataria como exceção.

Para a maior parte dos ambientes, você vai acabar escolhendo entre duas opções:

- Fazer o **upgrade in-place** em VM compatível e bem validada;
- Ou subir uma **nova VM** já na versão alvo e migrar a carga.

Se o ambiente for mais sensível, mais crítico ou mais moderno, a segunda opção quase sempre te dá uma vida mais tranquila.

---

### Checklist

- [x] Passo 1 - Validar se a VM está em um cenário suportado
- [x] Passo 2 - Executar a Azure VM Windows OS Upgrade Assessment Tool
- [x] Passo 3 - Revisar requisitos do Windows 11 e compatibilidade da VM
- [x] Passo 4 - Garantir backup e possibilidade de rollback
- [x] Passo 5 - Iniciar o upgrade via Windows Update / Feature Update
- [x] Passo 6 - Acompanhar a reinicialização e o progresso da atualização
- [x] Passo 7 - Validar o sistema operacional e a operação após o upgrade
- [x] Passo 8 - Fazer limpeza e fechamento da mudança
- [x] Passo 9 - Revisar impacto no plano de controle do Azure
- [x] Passo 10 - Avaliar quando faz mais sentido criar uma nova VM

---

## Artigos

| Nome | Link |
| :----------------------------------------------------------: | :--------------------------------------------------------------------------------------------------------: |
| In-place upgrade for supported VMs running Windows in Azure (client) | [Acessar](https://learn.microsoft.com/pt-br/troubleshoot/azure/virtual-machines/windows/in-place-system-upgrade) |
| Azure VM Windows OS Upgrade Assessment Tool | [Acessar](https://learn.microsoft.com/pt-br/troubleshoot/azure/virtual-machines/windows/windows-vm-osupgradeassessment-tool) |
| Windows 11 requirements | [Acessar](https://learn.microsoft.com/en-us/windows/whats-new/windows-11-requirements) |
| Windows Enterprise multi-session FAQ - Azure | [Acessar](https://learn.microsoft.com/en-us/azure/virtual-desktop/windows-multisession-faq) |

---

## The End!

Este é o fim de mais um artigo em nosso blog!

A ideia aqui foi te mostrar que, sim, **é possível realizar um upgrade in-place de Windows Client em uma VM do Azure**, mas com algumas diferenças importantes quando comparamos com Windows Server.

No Client, o processo é mais simples, mais próximo do Windows Update tradicional e menos “engenheirado”. Por outro lado, ele exige bastante atenção com suporte, requisitos do Windows 11, tipo de VM, recursos de segurança e, principalmente, com o cenário onde essa máquina está inserida.

Se for uma VM compatível, bem validada e com backup garantido, o processo pode funcionar muito bem.

Agora, se o ambiente pedir mais previsibilidade, mais padronização ou menos risco, criar uma nova VM continua sendo uma baita escolha.

Tudo depende do contexto, da criticidade da carga e da estratégia operacional do seu ambiente.

Até a próxima!