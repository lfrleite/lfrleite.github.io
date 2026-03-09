---
#layout: post
title: "Atualizando o Windows Server em uma VM no Azureo sem surpresas"
date: 2026-03-09 08:00:00
categories: [Azure]
tags: [azure, windows-server, upgrade, in-place-upgrade, virtual-machine]
slug: 'upgrade-in-place-windows-server-azure-vm'
image:
  path: assets/img/005/001-windows-server-upgrade-azure.png
---

Fala PessoALL! Estava um pouco sumido né? Mas vamos seguir nossa jornada!

Quando falamos de **atualização** de um Windows Server dentro do Azure, muita gente ainda pensa que o único caminho é criar uma nova máquina, migrar tudo manualmente e depois descomissionar a antiga. *(até pouco tempo sim rsrs)*

Mas a Microsoft melhorou (e muito) esse processo de atualização, inclusive é altamente recomendada.

Imagine um cenário onde você já possui diversas VMs de produção, com aplicações instaladas, integrações funcionando e precisa manter a integridade de segurança e melhorar a compatibilidade com as novas aplicações onde se faz necessário elevar a versão do sistema operacional mantendo o máximo possível da estrutura existente, o **upgrade in-place** será um caminho bem interessante.

> Mas aqui tem um detalhe MUITO importante: no Azure, esse processo precisa ser feito do jeito certo. Não é só montar uma ISO e sair clicando em **Next**.
{: .prompt-warning } 

**Neste artigo, vamos realizar um *upgrade in-place de Windows Server em uma VM do Azure*, entendendo os pré-requisitos, os cuidados necessários, a criação da mídia de upgrade e a execução do processo de forma organizada.**

---

### Pre-requisitos:

- Possuir uma VM do Azure executando Windows Server em um caminho de upgrade suportado
- Validar previamente a matriz de upgrade da versão atual para a versão de destino
- Garantir espaço livre suficiente no disco do sistema operacional
- Utilizar **Managed Disks**
- Criar snapshot do disco do sistema operacional e, se necessário, dos discos de dados
- *(opcional)* Desabilitar temporariamente antivírus e firewall dentro do sistema operacional

---

## Mão na massa!

> Antes de iniciar qualquer alteração, eu recomendo responder 4 perguntas:
{: .prompt-info }

1 - O caminho de upgrade da sua versão atual para a versão desejada é suportado?  
2 - Essa VM realmente precisa ser mantida, ou faz mais sentido criar uma nova?  
3 - Você possui rollback rápido via snapshot e/ou backup?  
4 - Você está ciente de que, após o upgrade in-place, a VM deixa de aproveitar algumas capacidades ligadas ao modelo original da imagem no Azure?

**Se essas respostas estiverem claras, aí sim seguimos.**

#### Passo 1

A primeira tarefa é validar se a sua origem e o seu destino fazem parte de um **caminho de upgrade suportado**.

Na prática, não é porque a VM está no Azure que qualquer salto de versão será permitido. O ideal é sempre consultar a matriz oficial antes de começar.

Além disso, vale um ponto importante: para Windows Server 2025, a Microsoft ampliou os caminhos suportados e permite upgrades mais longos em sistemas não clusterizados.

![winserver-upgrade](assets/img/005/002-windows-server-upgrade-azure.png){: .shadow .rounded-10}

> Antes de qualquer clique, valide a matriz oficial de upgrade. Começar um projeto sem essa checagem é pedir retrabalho. 
{: .prompt-warning }

#### Passo 2

Agora execute a ferramenta **Azure VM Windows OS Upgrade Assessment Tool**.

Essa ferramenta ajuda a validar o caminho de upgrade e possíveis problemas conhecidos antes de você iniciar a mudança de verdade.

Esse é o tipo de etapa que muita gente ignora, mas que faz total diferença quando o objetivo é evitar surpresa no meio da janela de manutenção.

#### Passo 3

Com o caminho validado, verifique o básico da VM:

* disco do sistema com espaço livre suficiente
* uso de **Managed Disks**
* sistema operacional saudável
* conectividade administrativa funcionando
* ausência de bloqueios por antivírus, antispyware ou firewall local

Se a VM ainda não estiver usando **Managed Disks**, esse é um requisito para seguir com o processo no Azure.

#### Passo 4

Agora vem uma das etapas mais importantes: **proteger o rollback**.

Crie snapshot do:

* disco do sistema operacional
* discos de dados, se houver

Essa etapa é o que vai te salvar caso o upgrade falhe no meio do processo ou o sistema não volte de forma saudável.

![ws-upgrade-002](assets/img/004/102-windows-server-upgrade-snapshot.png){: .shadow .rounded-10}

> Upgrade in-place sem snapshot é aposta. E ambiente corporativo não combina com aposta. {: .prompt-danger }

#### Passo 5

No Azure, o processo oficial para Windows Server utiliza uma **mídia de upgrade em formato de Managed Disk**, criada a partir de uma imagem especial do Marketplace.

Abaixo está um exemplo de script para criar o disco de mídia de upgrade.  
Adapte os parâmetros de acordo com o seu ambiente:

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


Passo 6

Com o disco criado, anexe essa mídia de upgrade na VM.

No portal do Azure:

abra a VM

vá em Disks

clique em Attach existing disks

selecione o disco de upgrade criado anteriormente

salve a configuração

Esse disco precisa estar na mesma região da VM e, se a VM estiver em zona, na mesma zona também.

Passo 7

Agora sim, com a VM em execução, conecte-se via RDP ou Azure Bastion, descubra a letra da unidade onde a mídia foi anexada e inicie o setup.

Para Windows Server 2016, 2019, 2022 ou 2025, você pode utilizar o seguinte comando:

.\setup.exe /auto upgrade /dynamicupdate disable /eula accept

Esse comando ajuda a automatizar o processo e evita travas por aceite manual do contrato de licença.

Se você estiver lidando especificamente com um alvo Windows Server 2012, o fluxo é mais manual e o setup é iniciado apenas com:

.\setup.exe

Depois, basta seguir o assistente de instalação e escolher a opção de manter arquivos, configurações e aplicações.

{: .shadow .rounded-10}

Passo 8

Durante o processo, a VM vai reiniciar e sua sessão RDP será desconectada automaticamente.

Nesse momento, você pode acompanhar o progresso por meio da funcionalidade de screenshot no portal do Azure, o que é muito útil quando a VM está em uma etapa em que o acesso remoto ainda não voltou.

Passo 9

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

Passo 10

Agora vem um alerta importantíssimo: mesmo com o upgrade concluído, a VM continua mantendo as informações da imagem original no Azure.

Ou seja, o sistema operacional foi atualizado, mas as propriedades da imagem no portal — como publisher, offer e plan — permanecem com os dados antigos.

Além disso, esse tipo de upgrade faz com que a VM deixe de aproveitar recursos como:

Auto guest patching

Auto OS image upgrades

Hotpatching

Azure Update Manager

Se você depende fortemente dessas capacidades, o caminho mais limpo continua sendo criar uma nova VM já na versão alvo.



O upgrade in-place resolve muito bem o sistema operacional, mas não “transforma” a VM em uma nova imagem nativa do Azure. {: .prompt-warning }



Checklist

Passo 1 - Validar o caminho de upgrade suportado

Passo 2 - Executar o OS Upgrade Assessment Tool

Passo 3 - Verificar espaço livre, saúde da VM e uso de Managed Disks

Passo 4 - Criar snapshot do disco do sistema e dos discos de dados

Passo 5 - Criar a mídia de upgrade em formato de Managed Disk

Passo 6 - Anexar o disco de upgrade na VM

Passo 7 - Executar o setup do Windows Server

Passo 8 - Acompanhar o progresso e aguardar reinicializações

Passo 9 - Validar o sistema após a atualização

Passo 10 - Remover artefatos temporários e revisar impactos no Azure

Artigos

[In-place upgrade for VMs running Windows Server in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows-in-place-upgrade)
[Overview of Windows Server upgrades](https://learn.microsoft.com/en-us/windows-server/get-started/upgrade-overview)

The End!

Este é o fim de mais um artigo em nosso blog!

A ideia aqui foi te mostrar que, sim, é possível realizar um upgrade in-place de Windows Server em uma VM do Azure de forma suportada e organizada, desde que você respeite o caminho de upgrade, proteja rollback e entenda os impactos operacionais no Azure.

Em alguns cenários, esse caminho faz muito sentido. Em outros, criar uma nova VM continua sendo o mais limpo.

Tudo depende do contexto, da criticidade da carga e da estratégia de operação do ambiente.

Até a próxima!