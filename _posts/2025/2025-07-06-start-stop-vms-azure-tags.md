---
#layout: post
title: "Como realizar um Start/Stop em VMs no Azure fora do horário de uso com TAGs"
date: 2025-07-06 12:00:00
categories: [Azure]
tags: [azure, tag, vm, windows, linux]
slug: 'azure-tag-start-stop-vms'
image:
  path: assets/img/001/001-start-stop.png
---

Fala pessoALL! Espero que vocês estejam bem! Seja bem vindos ao meu novo Blog pessoal técnico!

Estou extremamente feliz por publicar meu primeiro artigo e espero que vocês também gostem!

Uma das modalidades em alta na atualidade é o FinOps quando falamos de Tecnologia em Cloud.

FinOps é como o GPS das finanças na nuvem. Sabe aquela sensação de liberdade de usar a nuvem para tudo, mas sem ter noção de quanto está gastando? Pois é, FinOps é o super-herói que entra em cena para evitar surpresas na conta no final do mês.

Utilizando um Automation Account, você possui a liberdade de realizar um gatilho (trigger) forçando uma ou mais máquinas virtuais (VMs) a estarem realizando um stop (parada) total fora do horário comercial e um start (início) durante o horário comercial!

**Neste artigo, iremos mostrar como criar, configurar e utilizar um Runbook dentro de um Automation Account corretamente. Guiaremos passo a passo para garantir que sejam executados em suas VMs com as TAGs “START” ou “STOP” apenas quando necessário, economizando tempo e recursos.**

---

### Pre-requisitos:
- Possuir permissão no mínimo **Contributor** da Subscription

---

## Mão na massa!

Primeiramente vamos criar um Automation Account na assinatura de forma bem simples (***quase um NNF – Next > Next > Finish rsrs***)

#### Passo 1

1 - Na barra de busca vamos escrever por “Automation” e logo aparecerá o recurso Automation Account para selecionarmos:

![automation-account-create](/assets/img/001/002-start-stop.png){: .shadow .rounded-10} 
<br>

2 - Lembramos que para melhores práticas devemos sempre utilizar as definições do **Cloud Adoption Framework** (CAF) para gerar corretamente os nomes dos recursos, tanto para uma melhor organização como também praticidade em um ambiente com muitos recursos:

![automation-account-create](/assets/img/001/003-start-stop.png){: .shadow .rounded-10} 
<br>

> **O Automation Account precisa estar na mesma região que as VMs**
{: .prompt-warning } 
<br>

3 - Em “**Advanced**” vamos deixar selecionado a opção de “**System Assigned**”, para ser criado e habilitado a identidade gerenciada deste recurso:

![automation-account-create](/assets/img/001/004-start-stop.png){: .shadow .rounded-10} 
<br>
**Em seguida é finalizar e acessar o Automation Account criado para a próxima etapa!**

4 - Com o Automation Account devidamente criado, vamos em ‘**Runbooks**‘ para iniciar a configuração do nosso runbook de Start/Stop:

![runbook-create](/assets/img/001/005-start-stop.png){: .shadow .rounded-10} 
<br>

---

### Passo 2

Dentro de Runbooks podemos verificar que já existem 2 modelos oferecidos pelo Azure para criar a partir deles, mas já iremos disponibilizar um devidamente configurado com as informações completas pra uso.

1 - Dentro de Runbooks basta clicar em ‘**+ Create**‘:

![runbook-create](/assets/img/001/006-start-stop.png){: .shadow .rounded-10} 
<br>

2 - Em Runbook type, selecione **PowerShell**, e em seguida em Runtime Environment clique em ‘Select from existing‘:

![runbook-create](/assets/img/001/007-start-stop.png){: .shadow .rounded-10} 
<br>

Selecione ‘**PowerShell-7.2**‘:

![runbook-create](/assets/img/001/008-start-stop.png){: .shadow .rounded-10} 
<br>

> Pode ser que no momento que estás sendo executado, a tela pode ter sido alterada! A Microsoft adora mudar o layout do Microsoft Azure.
{: .prompt-info } 
<br>

3 - Como boas práticas é sempre importante inserir uma descrição, pois em um Automation Account você pode ter mais de 1 Runbook para diferentes tarefas:

![runbook-create](/assets/img/001/009-start-stop.png){: .shadow .rounded-10} 
<br>

4 - Agora com nosso Runbook criado seremos direcionados para editarmos o nosso Runbook PowerShell totalmente zerado. Logo abaixo deixarei o script para que possam copiar e colar diretamente no portal:

[Script Start/Stop VMs](https://github.com/lfrleite/Ruiz-Online/blob/main/Start-Stop-VMs/Start%20Stop%20VMs%20-%20Azure.md)
<br>

![runbook-create](/assets/img/001/010-start-stop.png){: .shadow .rounded-10} 
<br>

5 - Em seguida vamos clicar em ‘**Save**‘ em ‘**Publish**‘ para publicar o Runbook customizado que acabamos de criar.

---

### Passo 3

Nesse momento temos 50% do processo pronto! Agora vamos precisar ajustar a permissão do Automation Account para realizar as devidas ações nas VMs.

1 - Retorne no nível do Automation Account clicando no nome logo acima e acesse o menu ‘**Identity**‘ localizado em Account Settings:

![runbook-create](/assets/img/001/011-start-stop.png){: .shadow .rounded-10} 
<br>

2 - Na tela do ‘**System assigned**‘ você irá identificar a identidade gerenciada que criamos no começo junto com o Automation Account. Clique em ‘**Azure role assignments**‘ localizado logo abaixo do ID do objeto:

![runbook-create](/assets/img/001/012-start-stop.png){: .shadow .rounded-10} 
<br>

3 - Nesse momento precisamos ser o mais preciso possível pensando sempre no menor privilégio necessário para essa ação, que no caso seria realizar um Start e Stop em VMs. Para essa finalidade utilizaremos a função de RBAC ‘**Virtual Machine Contributor**‘ com o ‘Scope‘ diretamente na **Subscription**:

![runbook-create](/assets/img/001/013-start-stop.png){: .shadow .rounded-10} 
<br>
![runbook-create](/assets/img/001/014-start-stop.png){: .shadow .rounded-10} 
<br>

---

### Passo 4

Antes de aplicarmos em produção, seguindo as boas práticas, devemos sempre fazer testes antes. E por isso o Runbook possui uma função incrível onde pode ser feito o teste diretamente na ferramenta antes de agendarmos em produção!

1 - Vamos retornar ao Runbook ‘**Start-Stop-VMs**‘, clicamos em ‘**Edit > Edit in Portal**‘ para acessarmos o ambiente de testes:

![runbook-create](/assets/img/001/015-start-stop.png){: .shadow .rounded-10} 
<br>

2 - Em seguida clicaremos em ‘**Test Plan**‘ para acessar o ambiente de teste deste Runbook:
![runbook-create](/assets/img/001/016-start-stop.png){: .shadow .rounded-10} 
<br>

3 - Nesse momento vamos fazer um teste nas VMs para que possam **LIGAR** com as seguintes TAGs:

- TAGNAME = **Start**
- TAGVALUE = **08:00**
- SHUTDOWN = **False**

![runbook-create](/assets/img/001/017-start-stop.png){: .shadow .rounded-10} 
<br>

4 - Iremos abrir uma nova guia clicando com o botão direito em **Virtual Machines** ao lado esquerdo para avaliarmos e acompanhar o processo de testes:

![runbook-create](/assets/img/001/018-start-stop.png){: .shadow .rounded-10} 
<br>

> Como podem notar, todas as VMs já possuem as TAGs correspondentes e estão TODAS desalocadas **(Stoppadas)**.
{: .prompt-info } 
<br>

5 - Agora retornaremos a outra guia de teste e selecionamos ‘**Start**‘:

![runbook-create](/assets/img/001/019-start-stop.png){: .shadow .rounded-10} 
<br>
![runbook-create](/assets/img/001/020-start-stop.png){: .shadow .rounded-10} 
<br>

> Todas as VMs inicializaram com sucesso em nosso teste!
{: .prompt-info } 
<br>

6 - Vamos realizar o teste **reverso** agora desligando/desalocando as VMs com as seguintes TAGs:
- TAGNAME = **Stop**
- TAGVALUE = **18:00**
- SHUTDOWN = **True**

![runbook-create](/assets/img/001/021-start-stop.png){: .shadow .rounded-10} 
<br>

7 - Após as alterações realizadas, basta clicar novamente em ‘**Start**‘ para iniciar o teste reverso:

![runbook-create](/assets/img/001/022-start-stop.png){: .shadow .rounded-10} 
<br>
![runbook-create](/assets/img/001/023-start-stop.png){: .shadow .rounded-10} 
<br>

> E mais uma vez tivemos um resultado esperado, onde todas as VMs com as TAGs ‘**Stop : 18:00**‘ foram desalocadas com sucesso!
{: .prompt-info } 
<br>

---

### Passo 5

Por último, mas não menos importante, iremos realizar o agendamento (_***Schedule***_) para que a função de Start/Stop funcione de forma automatizada.

1 - Retornamos ao Runbook, clicaremos em ‘**Schedules**‘ e em seguida ‘**Add a schedule**‘:

![runbook-schedule](/assets/img/001/024-start-stop.png){: .shadow .rounded-10} 
<br>
![runbook-schedule](/assets/img/001/025-start-stop.png){: .shadow .rounded-10} 
<br>

2 - Clicamos em ‘**Add a schedule**‘ novamente para configurarmos o primeiro agendamento de START das VMs:

1. Campo **Name**: Inclua um nome para o agendamento. Ex.: Start-VMs-08-00
2. **Descrição**: Como mencionado anteriormente, este é um campo importante e livre para a descrição do motivo deste agendamento.
3. Campo ‘**Starts**‘: Você pode determinar quando este agendamento começará a valer de verdade! O horário precisa refletir EXATAMENTE o horário de inicio do START. Por melhores práticas o ideal é deixar o agendamento para a data que foi acordado com o cliente mediante uma GMUD.
4. **Time Zone**: Aqui você determina em qual GMT ou UTC o agendamento ocorrerá. É um dos campos mais importantes do agendamento, portanto precisa ser o mais assertivo possível na escolha.
5. 6 e 7 – **Recorrência**: Outro fator importante: “Qual será a recorrência que este agendamento ocorrerá? Por hora? Por dia? Por semana? Por mês?“. Este é o ponto chave onde podemos dizer em quais dias da semana irá acontecer este agendamento. No nosso caso, selecionamos a recorrência de forma SEMANAL durante o dia útil (Segunda à Sexta).

![runbook-schedule](/assets/img/001/026-start-stop.png){: .shadow .rounded-10} 
<br>

**Por fim é só clicar em Create e concluir a primeira parte do agendamento!**

> Muito bem agora temos o primeiro agendamento, mas agendamento de que? Pois é, ainda não temos os parâmetros necessários para que este agendamento de fato esteja totalmente funcional.
{: .prompt-info }
<br>

3 - Vamos configurá-lo clicando em ‘**Parameters and run settings**‘:

![runbook-schedule](/assets/img/001/027-start-stop.png){: .shadow .rounded-10} 
<br>

4 - Nesse momento vamos reescrever as informações que fizemos no teste anterior, para que as VMs possam **LIGAR** com as seguintes TAGs:
- TAGNAME = **Start**
- TAGVALUE = **08:00**
- SHUTDOWN = **False**

![runbook-schedule](/assets/img/001/028-start-stop.png){: .shadow .rounded-10} 
<br>

Só clicar em OK e pronto! Nosso primeiro agendamento está pronto! Agora vamos partir para o segundo agendamento para desligar as VMs às 18:00.

> Como os passos são completamente iguais ao anterior, não será necessário ir passo a passo, pois temos a certeza que você já sabe 😉
{: .prompt-tip } 
<br>

5 - Vamos reescrever as informações de forma reversa, agora **desligando/desalocando** as VMs com as seguintes TAGs:
- TAGNAME = **Stop**
- TAGVALUE = **18:00**
- SHUTDOWN = **True**

![runbook-schedule](/assets/img/001/029-start-stop.png){: .shadow .rounded-10} 
<br>

E finalmente temos um Automation Account devidamente configurado, com os agendamentos definidos e os parâmetros corretos para o perfeito funcionamento:

![runbook-schedule](/assets/img/001/030-start-stop.png){: .shadow .rounded-10} 
<br>

> Agora é só aguardar o dia e horário agendado para que a ‘*mágica*‘ aconteça!

6 - Podemos validar o funcionamento através do recurso ‘**Jobs**‘:

![runbook-schedule](/assets/img/001/031-start-stop.png){: .shadow .rounded-10} 
<br>
![runbook-schedule](/assets/img/001/032-start-stop.png){: .shadow .rounded-10} 
<br>

---

> Pontos importantes que valem a pena ser citado
{: .prompt-danger } 

- O Runbook executa a função por meio de Filas (QUEUE), ou seja, ele irá enfileirar as VMs e ir executando **uma por uma** conforme os agendamentos configurados;
- Dependendo do quantitativo de máquinas virtuais existentes em uma assinatura, o ideal é ir quebrando as execuções por Automation Accounts;
- A Microsoft recomenda o uso de Logic App para estas funções, contudo dependendo do consumo pode chegar a um valor considerável, por isso é importante validar o quantitativo no ambiente e segmentar.

---

### Checklist
- [x] Passo 1 - Criando um Automation Account
- [x] Passo 2 - Criando um Runbook
- [x] Passo 3 - Habilitando uma identidade gerenciada e fornecendo os privilégios necessários
- [x] Passo 4 - Testando o agendamento
- [x] Passo 5 - Aplicando em Produção

---

## Inspiração

**Esse artigo foi inspirado no vídeo do Raphael Andrade**:

<iframe width="800" height="512" src="https://www.youtube.com/embed/hYANqXSxVMU" title="Como economizar até 50% no custo de VMs com Start-Stop automático" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## Artigos

| Nome                                                        | Link                                                                                            |
| :-----------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| Tipos de runbook da Automação do Azure                      | <https://learn.microsoft.com/pt-br/azure/automation/automation-runbook-types?tabs=lps72%2Cpy10> |
| Habilitar uma Identidade Gerenciada em um Automation Account| <https://learn.microsoft.com/pt-br/azure/automation/enable-managed-identity-for-automation>     |
| Script PowerShell                                           | <https://github.com/lfrleite/Ruiz-Online/blob/main/Start%20Stop%20VMs%20-%20Azure.md>           |

---

## The End!

Este é o fim deste nosso primeiro artigo do nosso blog! Espero que tenham gostado e possam aplicar em seus clientes! Até a próxima!

---

> "Confie no Senhor de todo o coração, não dependa do seu próprio entendimento. Busque a vontade dEle em **TUDO** que fizer, e Ele lhe mostrará o caminho que deve seguir" - PV 3:5-6