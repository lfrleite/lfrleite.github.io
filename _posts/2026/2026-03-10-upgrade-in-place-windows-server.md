---
#layout: post
title: "Atualizando o Windows Server em uma VM do Azure sem surpresas"
date: 2026-03-11 15:00:00 -03:00
categories: [Azure]
tags: [azure, windows-server, upgrade, in-place-upgrade, virtual-machine]
slug: 'upgrade-in-place-windows-server-azure-vm'
image:
  path: assets/img/005/001-windows-server-upgrade-azure.png
---

Fala PessoALL! Estava um pouco sumido né? Mas agora estamos com a nossa programação normal de volta!

Quando falamos de atualização de um Windows Server dentro do Azure, muita gente ainda pensa que o único caminho é criar uma nova máquina, migrar tudo manualmente e depois descomissionar a antiga. E sendo bem sincero, em muitos cenários esse ainda continua sendo um excelente caminho.

Mas nem sempre ele é o mais prático.

Imagine aquele ambiente onde você já tem uma VM pronta, com aplicação instalada, integrações funcionando, acessos configurados, ajustes finos feitos ao longo do tempo e uma janela de manutenção relativamente curta. Nesses casos, recriar tudo do zero pode gerar muito retrabalho. É justamente aqui que o **upgrade in-place** pode fazer bastante sentido.

Hoje a Microsoft já possui um processo suportado para realizar **upgrade in-place em VMs Windows Server no Azure**, permitindo elevar a versão do sistema operacional sem necessariamente reconstruir toda a máquina do zero.

Mas como tudo em infraestrutura, isso precisa ser feito com critério.

Neste artigo, vamos realizar um **upgrade in-place de Windows Server em uma VM do Azure**, entendendo os pré-requisitos, os cuidados necessários, a criação da mídia de upgrade e a execução do processo de forma organizada, prática e suportada.

---

## Quando esse método faz sentido?

Antes de sair clicando em tudo, vale alinhar expectativa.

O upgrade in-place costuma fazer bastante sentido quando você tem:

- uma VM já em produção com aplicações configuradas;
- integrações que dariam trabalho para recriar;
- necessidade de reduzir tempo de rebuild;
- uma janela de mudança menor;
- necessidade de atualizar o sistema operacional mantendo a estrutura atual.

Agora, se a sua intenção é aproveitar todos os recursos mais modernos nativos do Azure, padronizar tudo do zero e já sair com uma VM “nova de verdade”, muitas vezes o melhor caminho ainda continua sendo criar uma nova VM e migrar a carga.

E esse ponto é importante, porque o upgrade in-place atualiza o sistema operacional, mas **não transforma a VM em uma imagem nova nativa do Azure**. Ou seja, ela continua carregando algumas características da imagem original.

---

### Pre-requisitos

Antes de começar, valide os seguintes pontos:

- Possuir permissão de no mínimo **Contributor** na Subscription ou no Resource Group;
- Confirmar se a **origem e o destino** fazem parte de um caminho de upgrade suportado;
- Utilizar **Managed Disks**, pois outros tipos não são suportados neste processo;
- Garantir espaço livre suficiente no disco do sistema operacional;
- Criar snapshot do disco do sistema operacional e, se houver, também dos discos de dados;
- Validar se a VM está preparada para **Windows Server volume licensing**;
- Considerar o idioma da VM, pois a mídia especial de upgrade é criada em **en-US**;
- Opcionalmente, desabilitar temporariamente antivírus, antispyware e firewall durante a janela de manutenção.

> **A Microsoft também recomenda a execução da ferramenta de assessment antes do processo para validar compatibilidade e possíveis impedimentos.** 
{: .prompt-info }

---

## Mão na massa!

---

#### Passo 1

A primeira tarefa é confirmar se a sua origem e o seu destino fazem parte de um caminho de upgrade suportado.

Na prática, não é porque a VM está no Azure que qualquer salto de versão será permitido. O ideal é sempre consultar a matriz oficial antes de começar, principalmente se você estiver trabalhando com versões mais antigas ou planejando um salto maior.

Hoje, para cenários não clusterizados, o Windows Server 2025 aceita upgrade direto a partir do **Windows Server 2012 R2 e versões posteriores**. Já para outras versões, a recomendação continua sendo validar a matriz oficial antes da execução.

**[INSERIR IMAGEM DA MATRIZ OFICIAL DE UPGRADE]**

> **Em ambiente real, esse é o tipo de etapa que evita retrabalho antes mesmo da janela de manutenção começar.** 
{: .prompt-info }

---

#### Passo 2

A Microsoft disponibiliza uma ferramenta chamada **Azure VM Windows OS Upgrade Assessment Tool**, que ajuda a validar a compatibilidade da máquina com o processo de upgrade in-place.

Essa ferramenta é útil para identificar previamente:

- caminho de upgrade suportado;
- possíveis incompatibilidades;
- problemas de preparação do ambiente;
- limitações que podem impedir a atualização.

A ideia aqui é simples: quanto mais problema você encontrar antes, menos chance de transformar a janela de mudança em dor de cabeça.

1. Acesse a documentação oficial da ferramenta de avaliação e realize o download diretamente do repositório oficial na VM que será atualizada;
2. Execute a ferramenta e valide o resultado apresentado.

**[INSERIR IMAGEM DA FERRAMENTA DE ASSESSMENT]**

> **Caso a ferramenta identifique alguma falha, corrija antes de prosseguir.** 
{: .prompt-warning }

---

#### Passo 3

Agora vamos validar alguns pontos que parecem simples, mas que fazem bastante diferença no sucesso do processo.

1. **Confirme se a VM utiliza Managed Disks**, pois esse processo aceita somente esse modelo;
2. **Verifique se há espaço livre suficiente no disco do sistema operacional**. Não é o tipo de validação para fazer “no limite”; trabalhe sempre com folga;
3. **Valide o tipo de licenciamento da VM**, pois a mídia especial de upgrade fornecida pelo Azure exige que a máquina esteja preparada para **Windows Server volume licensing**;
4. **Revise o idioma do sistema**, pois a mídia especial de upgrade utilizada nesse processo é criada em **en-US**. Em ambientes fora desse padrão, vale validar isso com atenção antes de avançar.

> **Esse é um daqueles pontos que parecem detalhe... até o dia em que travam a mudança.** 
{: .prompt-warning }

---

#### Passo 4

Como todo administrador, você precisa pensar sempre na proteção e restauração em caso de algum problema durante o processo.

Nesta etapa iremos criar snapshot do disco do sistema operacional e, caso existam, também dos discos de dados. Esse procedimento é importante para possibilitar rollback caso o upgrade falhe ou a VM não volte de forma saudável.

1. Acesse a VM no portal do Azure;
2. Vá até a seção de discos;
3. Crie snapshot do disco do sistema operacional;
4. Caso existam discos de dados, crie snapshot deles também.

**[INSERIR IMAGEM DA TELA DE SNAPSHOT]**

> **Upgrade in-place sem snapshot é aposta. E ambiente corporativo não combina com aposta.** 
{: .prompt-danger }

> **Caso você tenha sido designado para realizar upgrade de mais de 5 / 10 / 50 VMs, aqui no blog já temos um artigo que lhe ajudará a criar snapshots de forma automatizada:** [Criando Snapshots de VMs no Azure com TAGs](/posts/criando-snapshot-de-vms-atraves-de-tags/) 
{: .prompt-info }

---

#### Passo 5

No Azure, o processo oficial para Windows Server utiliza uma mídia de upgrade em formato de **Managed Disk**, criada a partir de uma imagem especial do Marketplace.

Essa imagem utiliza como base:

- **Publisher:** `MicrosoftWindowsServer`
- **Offer:** `WindowsServerUpgrade`
- **SKU:** varia conforme a versão de destino, como por exemplo `server2025Upgrade`, `server2022Upgrade`, `server2019Upgrade`, `server2016Upgrade` ou `server2012Upgrade`

Abaixo está um exemplo de script em PowerShell para criar esse disco de mídia de upgrade.

1. Abra o **Cloud Shell** no portal do Azure;
2. Ajuste os parâmetros conforme a sua necessidade;
3. Execute o código abaixo e aguarde o fim da criação do disco.

```powershell
# Resource Group onde o disco de upgrade será criado
$resourceGroup = "WindowsServerUpgrades"

# Mesma região da VM que será atualizada
$location = "BrazilSouth"

# Zona da VM, se existir. Para VMs regionais, use ""
$zone = ""

# Nome do disco de upgrade
$diskName = "WindowsServer2025UpgradeDisk"

# SKU da mídia de upgrade
# Opções:
# server2025Upgrade
# server2022Upgrade
# server2019Upgrade
# server2016Upgrade
# server2012Upgrade
$sku = "server2025Upgrade"

# Parâmetros comuns
$publisher = "MicrosoftWindowsServer"
$offer = "WindowsServerUpgrade"
$managedDiskSKU = "Standard_LRS"

# Obtém a versão mais recente da imagem especial de upgrade
$versions = Get-AzVMImage -PublisherName $publisher -Location $location -Offer $offer -Skus $sku |
    Sort-Object -Descending {[version] $_.Version}

$latestString = $versions[0].Version

# Obtém a imagem
$image = Get-AzVMImage -Location $location `
                       -PublisherName $publisher `
                       -Offer $offer `
                       -Skus $sku `
                       -Version $latestString

# Cria o Resource Group, se necessário
if (-not (Get-AzResourceGroup -Name $resourceGroup -ErrorAction SilentlyContinue)) {
    New-AzResourceGroup -Name $resourceGroup -Location $location
}

# Cria a configuração do disco
if ($zone) {
    $diskConfig = New-AzDiskConfig -SkuName $managedDiskSKU `
                                   -CreateOption FromImage `
                                   -Zone $zone `
                                   -Location $location
}
else {
    $diskConfig = New-AzDiskConfig -SkuName $managedDiskSKU `
                                   -CreateOption FromImage `
                                   -Location $location
}

Set-AzDiskImageReference -Disk $diskConfig -Id $image.Id -Lun 0

# Cria o disco de upgrade
New-AzDisk -ResourceGroupName $resourceGroup `
           -DiskName $diskName `
           -Disk $diskConfig
```

[INSERIR IMAGEM DO CLOUD SHELL / DISCO CRIADO]

> Esse disco deve ser criado na mesma região da VM e, se a VM estiver em zona, preferencialmente na mesma zona também. 
{: .prompt-info }


#### Passo 6

Com o disco criado, anexe essa mídia de upgrade na VM.

No portal do Azure, abra a VM;

Vá em Disks;

Clique em Attach existing disks;

Selecione o disco de upgrade criado anteriormente;

Salve a configuração.

[INSERIR IMAGEM DO ANEXO DO DISCO NA VM]

> Esse disco precisa estar na mesma região da VM e, se a VM estiver em zona, na mesma zona também. 
{: .prompt-info }

---

#### Passo 7

Agora sim, com a VM em execução, conecte-se via RDP ou Azure Bastion, descubra a letra da unidade onde a mídia foi anexada e inicie o setup.

Acesse a VM via RDP ou Bastion;

Abra o PowerShell como Administrador;

Navegue até a unidade onde o disco de upgrade foi montado.

Para Windows Server 2016, 2019, 2022 ou 2025, você pode utilizar o seguinte comando:

```powershell
.\setup.exe /auto upgrade /dynamicupdate disable /eula accept
```

Esse comando ajuda a automatizar o processo e evita travas por aceite manual do contrato de licença.

Ou, se você estiver lidando especificamente com um alvo Windows Server 2012, o fluxo é mais manual e o setup é iniciado apenas com:

```powershell
.\setup.exe
```

Depois disso, basta seguir o assistente de instalação e escolher a opção de manter arquivos, configurações e aplicações, quando aplicável ao cenário suportado.

[INSERIR IMAGEM DO SETUP SENDO EXECUTADO]

> Selecione com atenção a imagem correta de destino conforme a versão atual e a matriz de upgrade suportada. 
{: .prompt-warning }

---

#### Passo 8

Durante o processo, a VM vai reiniciar e sua sessão RDP será desconectada automaticamente.

Isso é esperado.

Nesse momento, você pode acompanhar o progresso por meio da funcionalidade de screenshot / boot diagnostics no portal do Azure, o que é muito útil quando a VM ainda não voltou para acesso remoto.

[INSERIR IMAGEM DO BOOT DIAGNOSTICS / SCREENSHOT DA VM]

---

#### Passo 9

Concluído o upgrade, valide cuidadosamente o ambiente.

Verifique a versão do Windows Server;

Valide serviços e aplicações instaladas;

Confirme a conectividade de rede;

Revise roles e features;

Verifique o status de ativação/licenciamento;

Confirme a integridade geral da VM.

Depois disso, faça a limpeza dos artefatos temporários:

remova o disco de mídia de upgrade;

reabilite antivírus, antispyware e firewall, caso tenham sido desabilitados;

mantenha os snapshots por uma janela segura e remova-os quando não forem mais necessários.

> Não trate “subiu o Windows” como sinônimo de “mudança concluída”. O que precisa voltar é a operação. 
{: .prompt-warning }

---

#### Passo 10

Agora vem um alerta importantíssimo: mesmo com o upgrade concluído com sucesso, a VM continua mantendo as informações da imagem original no Azure.

Ou seja, o sistema operacional foi atualizado, mas as propriedades da imagem no portal, como publisher, offer e plan, permanecem com os dados antigos.

Além disso, esse tipo de upgrade faz com que a VM deixe de aproveitar recursos como:

Auto guest patching;

Auto OS image upgrades;

Hotpatching;

Azure Update Manager.

Se você depende fortemente dessas capacidades, o caminho mais limpo continua sendo criar uma nova VM já na versão alvo e migrar a carga.

> O upgrade in-place resolve muito bem o sistema operacional, mas não “transforma” a VM em uma nova imagem nativa do Azure. 
{: .prompt-warning }

---

## Checklist

- [x] Passo 1 - Validar o caminho de upgrade suportado
- [x] Passo 2 - Executar o OS Upgrade Assessment Tool
- [x] Passo 3 - Verificar disco, licenciamento, idioma e uso de Managed Disks
- [x] Passo 4 - Criar snapshot do disco do sistema e dos discos de dados
- [x] Passo 5 - Criar a mídia de upgrade em formato de Managed Disk
- [x] Passo 6 - Anexar o disco de upgrade na VM
- [x] Passo 7 - Executar o setup do Windows Server
- [x] Passo 8 - Acompanhar o progresso e aguardar reinicializações
- [x] Passo 9 - Validar o sistema após a atualização
- [x] Passo 10 - Remover artefatos temporários e revisar impactos no Azure


## The End!

Este é o fim de mais um artigo em nosso blog!

A ideia aqui foi te mostrar que, sim, é possível realizar um upgrade in-place de Windows Server em uma VM do Azure de forma suportada e organizada, desde que você respeite o caminho de upgrade, proteja rollback e entenda os impactos operacionais no Azure.

Em alguns cenários, esse caminho faz muito sentido. Em outros, criar uma nova VM continua sendo o mais limpo.

Tudo depende do contexto, da criticidade da carga e da estratégia de operação do ambiente.

Até a próxima!

```powershell
---
#layout: post
title: "Atualizando o Windows Server em uma VM do Azure sem surpresas"
date: 2026-03-09 08:00:00 -03:00
categories: [Azure]
tags: [azure, windows-server, upgrade, in-place-upgrade, virtual-machine]
slug: 'upgrade-in-place-windows-server-azure-vm'
image:
  path: assets/img/005/001-windows-server-upgrade-azure.png
---

Fala PessoALL! Estava um pouco sumido né? Mas agora estamos com a nossa programação normal de volta!

Quando falamos de **atualização** de um Windows Server dentro do Azure, muita gente ainda pensa que o único caminho é criar uma nova máquina, migrar tudo manualmente e depois descomissionar a antiga. *(até pouco tempo sim rsrs)*

Mas a Microsoft melhorou (e muito) esse processo de atualização, inclusive é altamente recomendada.

Imagine um cenário onde você já possui diversas VMs de produção, com aplicações instaladas, integrações funcionando e precisa manter a integridade de segurança e melhorar a compatibilidade com as novas aplicações onde se faz necessário elevar a versão do sistema operacional mantendo o máximo possível da estrutura existente, o **upgrade in-place** será um caminho bem interessante.

**Neste artigo, vamos realizar um *upgrade in-place de Windows Server em uma VM do Azure*, entendendo os pré-requisitos, os cuidados necessários, a criação da mídia de upgrade e a execução do processo de forma organizada.**

---

### Pre-requisitos:

- Possuir permissão de no mínimo Contributor da Subscription;
- Validar previamente a matriz de upgrade da versão atual para a versão de destino - [Versões](https://learn.microsoft.com/pt-br/azure/virtual-machines/windows-in-place-upgrade#prerequisites);
- Garantir mínimo **32GB** de espaço livre suficiente no disco do sistema operacional;
- Somente **Managed Disks** é aceito, outro tipo não é suportado;
- Criar snapshot do disco do sistema operacional e, se houver disco de dados também;
- *(opcional)* Desabilitar temporariamente antivírus e firewall dentro do S.O.

---

## Mão na massa!

---

#### Passo 1

> Antes de qualquer clique, valide a matriz oficial de upgrade. 
{: .prompt-warning }

A primeira tarefa é validar se a sua origem e o seu destino fazem parte de um **caminho de upgrade suportado**.

Na prática, não é porque a VM está no Azure que qualquer salto de versão será permitido. O ideal é sempre consultar a matriz oficial antes de começar.

A própria documentação da Microsoft orienta

![winserver-upgrade](assets/img/005/002-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

***Opcional***
<br>
A Microsoft sugere que executemos **[Ferramenta de avaliação de atualização do sistema operacional Windows da VM do Azure](https://learn.microsoft.com/pt-br/troubleshoot/azure/virtual-machines/windows/windows-vm-osupgradeassessment-tool)** onde valida se o Sistema Operacional possui compatibilidade com o modelo de upgrade in-place. É uma ferramenta bem simples e de fácil utilização, onde você irá realizar o download da ferramenta diretamente do [repositório oficial no Github](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/Windows_OSUpgrade_Assessment_Validation) para a VM que será realizada essa atualização e executá-lo.

Caso ocorra alguma falha, ele retornará uma imagem parecida com essa:
![winserver-upgrade](assets/img/005/003-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

---

#### Passo 2 

Como todo administrador, você precisa pensar sempre na proteção e restauração em caso de algum problema durante o processo de atualização. Nessa etapa iremos realizar um snapshot do disco para (caso seja necessário) restaurá-lo em seguida.

> Caso você tenha sido designado para realizar um upgrade de mais de **5 / 10 / 50** VMs, nós temos um artigo que lhe auxiliará a realizar esses snapshots de forma automatizada! [Criando Snapshots de VMs no Azure com TAGs](https://blog.ruizsolutions.online/posts/criando-snapshot-de-vms-atraves-de-tags/). 
{: .prompt-info }
<br>

Essa etapa é o que vai te salvar caso o upgrade falhe no meio do processo ou o sistema não volte de forma saudável.

Imagem

> Upgrade in-place sem snapshot é aposta. E ambiente corporativo não combina com aposta. 
{: .prompt-danger }
<br>

---

#### Passo 5

No Azure, o processo oficial para Windows Server utiliza uma **mídia de upgrade em formato de Managed Disk**, criada a partir de uma imagem especial do Marketplace.

Abaixo está um exemplo de script para criar o disco de mídia de upgrade.  
Adapte os parâmetros de acordo com o seu ambiente. Sugerimos que copie e cole em um bloco de notas ou outro editor como VSCode pra melhorar a visualização e edição.

1 - Abra o Cloud Shell, copie e cole o código abaixo (após ter ajustado conforme a sua necessidade), e basta clicar em enter e aguardar o fim da configuração.

```powershell
# Resource group onde o disco de upgrade será criado
$resourceGroup = "WindowsServerUpgrades"

# Mesma região da VM que será atualizada
$location = "BrazilSouth"

# Zona da VM, se existir. Para VMs regionais, use ""
$zone = ""

# Nome do disco de upgrade
$diskName = "WindowsServer2025UpgradeDisk"

# SKU da mídia de upgrade
# Opções: server2025Upgrade, server2022Upgrade, server2019Upgrade, server2016Upgrade, server2012Upgrade
$sku = "server2025Upgrade"

# Parâmetros comuns
$publisher = "MicrosoftWindowsServer"
$offer = "WindowsServerUpgrade"
$managedDiskSKU = "Standard_LRS"

# Obtém a versão mais recente da imagem especial de upgrade
$versions = Get-AzVMImage -PublisherName $publisher -Location $location -Offer $offer -Skus $sku | Sort-Object -Descending {[version] $_.Version}
$latestString = $versions[0].Version

# Obtém a imagem
$image = Get-AzVMImage -Location $location `
                       -PublisherName $publisher `
                       -Offer $offer `
                       -Skus $sku `
                       -Version $latestString

# Cria o Resource Group, se necessário
if (-not (Get-AzResourceGroup -Name $resourceGroup -ErrorAction SilentlyContinue)) {
    New-AzResourceGroup -Name $resourceGroup -Location $location
}

# Cria a configuração do disco
if ($zone) {
    $diskConfig = New-AzDiskConfig -SkuName $managedDiskSKU `
                                   -CreateOption FromImage `
                                   -Zone $zone `
                                   -Location $location
} else {
    $diskConfig = New-AzDiskConfig -SkuName $managedDiskSKU `
                                   -CreateOption FromImage `
                                   -Location $location
}

Set-AzDiskImageReference -Disk $diskConfig -Id $image.Id -Lun 0

# Cria o disco de upgrade
New-AzDisk -ResourceGroupName $resourceGroup `
           -DiskName $diskName `
           -Disk $diskConfig
```

---

#### Passo 6

Com o disco criado, anexe essa mídia de upgrade na VM.

1 - No portal do Azure abra a VM, vá em Disks e clique em Attach existing disks. Selecione o disco de upgrade criado anteriormente e salve a configuração

> Esse disco precisa estar na mesma região da VM e, se a VM estiver em zona, na mesma zona também.
{: .prompt-warning }
<br>

---

#### Passo 7

Agora sim, com a VM em execução, conecte-se via RDP ou Azure Bastion, descubra a letra da unidade onde a mídia foi anexada e inicie o setup.

1 - Abra o PowerShell como Administrador e em seguida navegue até 

Para Windows Server **2016, 2019, 2022 ou 2025**, você pode utilizar o seguinte comando:

```powershell
.\setup.exe /auto upgrade /dynamicupdate disable /eula accept
```

Esse comando ajuda a automatizar o processo e evita travas por aceite manual do contrato de licença.

Ou se você estiver lidando especificamente com um alvo Windows Server **2012**, o fluxo é mais manual e o setup é iniciado apenas com:

```powershell
.\setup.exe
```

Depois, basta seguir o assistente de instalação e escolher a opção de manter arquivos, configurações e aplicações.

---

#### Passo 8

Durante o processo, a VM vai reiniciar e sua sessão RDP será desconectada automaticamente.

Nesse momento, você pode acompanhar o progresso por meio da funcionalidade de screenshot no portal do Azure, o que é muito útil quando a VM está em uma etapa em que o acesso remoto ainda não voltou.

---

#### Passo 9

Concluído o upgrade, valide:

versão do Windows Server

serviços e aplicações

conectividade

roles e features

status de ativação

integridade geral da VM

Depois disso, faça a limpeza dos artefatos temporários:

exclua snapshots antigos, se não forem mais necessários

remova o disco de mídia de upgrade

reabilite antivírus, antispyware e firewall, se eles tiverem sido desabilitados

---

#### Passo 10

Agora vem um alerta importantíssimo: mesmo com o upgrade concluído, a VM continua mantendo as informações da imagem original no Azure.

Ou seja, o sistema operacional foi atualizado, mas as propriedades da imagem no portal — como publisher, offer e plan — permanecem com os dados antigos.

Além disso, esse tipo de upgrade faz com que a VM deixe de aproveitar recursos como:

Auto guest patching

Auto OS image upgrades

Hotpatching

Azure Update Manager

Se você depende fortemente dessas capacidades, o caminho mais limpo continua sendo criar uma nova VM já na versão alvo.



O upgrade in-place resolve muito bem o sistema operacional, mas não “transforma” a VM em uma nova imagem nativa do Azure. {: .prompt-warning }



### Checklist

- [x] Passo 1 - Validar o caminho de upgrade suportado
- [x] Passo 2 - Executar o OS Upgrade Assessment Tool
- [x] Passo 3 - Verificar espaço livre, saúde da VM e uso de Managed Disks
- [x] Passo 4 - Criar snapshot do disco do sistema e dos discos de dados
- [x] Passo 5 - Criar a mídia de upgrade em formato de Managed Disk
- [x] Passo 6 - Anexar o disco de upgrade na VM
- [x] Passo 7 - Executar o setup do Windows Server
- [x] Passo 8 - Acompanhar o progresso e aguardar reinicializações
- [x] Passo 9 - Validar o sistema após a atualização
- [x] Passo 10 - Remover artefatos temporários e revisar impactos no Azure

---

## Inspiração

**Esse artigo foi inspirado no vídeo do Raphael Andrade**:

<iframe width="700" height="512" src="https://www.youtube.com/embed/-T7AAPQ9Sos" title="Como Atualizar o Windows Server no Azure | Upgrade In-Place" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## Artigos

| :---|:---|
| Nome | Link |
| :---|:---|
| In-place upgrade for VMs running Windows Server in Azure | <https://learn.microsoft.com/pt-br/azure/virtual-machines/windows-in-place-upgrade> |
| Overview of Windows Server upgrades | <https://learn.microsoft.com/pt-br/windows-server/get-started/upgrade-overview> |
| Requisitos de hardware do Windows Server | <https://learn.microsoft.com/pt-br/windows-server/get-started/hardware-requirements?tabs=storage&pivots=windows-server-2025>
| :---|:---|

---

## The End!

Este é o fim de mais um artigo em nosso blog!

A ideia aqui foi te mostrar que, sim, é possível realizar um upgrade in-place de Windows Server em uma VM do Azure de forma suportada e organizada, desde que você respeite o caminho de upgrade, proteja rollback e entenda os impactos operacionais no Azure.

Em alguns cenários, esse caminho faz muito sentido. Em outros, criar uma nova VM continua sendo o mais limpo.

Tudo depende do contexto, da criticidade da carga e da estratégia de operação do ambiente.

Até a próxima!
```