---
#layout: post
title: "Atualizando o Windows Server em uma VM do Azure sem surpresas"
date: 2026-03-16 12:00:00 -03:00
categories: [Azure]
tags: [azure, windows-server, upgrade, in-place-upgrade, virtual-machine]
slug: 'upgrade-in-place-windows-server-azure-vm'
image:
  path: assets/img/005/001-windows-server-upgrade-azure.png
---

Fala pessoALL! Estava um pouco sumido né? Mas agora estamos com a nossa programação normal de volta!

Quando falamos de atualização de um Windows Server de uma VM do Azure, muita gente ainda pensa que o único caminho é criar uma nova máquina, migrar tudo manualmente e depois descomissionar a antiga.

Imagine aquele ambiente onde você já tem uma VM pronta, com aplicação instalada, integrações funcionando, acessos configurados, ajustes finos feitos ao longo do tempo e uma janela de manutenção relativamente curta. Nesses casos, recriar tudo do zero pode gerar muito retrabalho. É justamente aqui que o **upgrade in-place** pode fazer bastante sentido.

Hoje a Microsoft já possui um processo suportado para realizar **upgrade in-place em VMs Windows Server no Azure**, permitindo elevar a versão do sistema operacional sem necessariamente reconstruir toda a máquina do zero.

**Neste artigo, vamos realizar um *upgrade in-place de Windows Server em uma VM do Azure*, entendendo os pré-requisitos, tomando todos os cuidados necessários, criando uma mídia temporária para o upgrade e a execução do processo de forma organizada, prática e suportada.**

---

## Quando esse método faz sentido?

Antes de sair clicando em tudo, precisamos alinhar as expectativa. O upgrade in-place costuma fazer bastante sentido quando você tem:

- Uma VM já em produção com aplicações configuradas;
- Integrações que dariam trabalho para recriar;
- Necessidade de reduzir tempo de rebuild e atualizar o sistema operacional mantendo a estrutura atual;

---

### Pre-requisitos

- Possuir permissão de no mínimo Contributor da Subscription;
- Validar previamente a matriz de upgrade da versão atual para a versão de destino - [Versões](https://learn.microsoft.com/pt-br/azure/virtual-machines/windows-in-place-upgrade#prerequisites);
- Garantir mínimo **32GB** de espaço livre suficiente no disco do sistema operacional;
- Somente **Managed Disks** é aceito, outro tipo não é suportado;
- Criar snapshot do disco do sistema operacional e, se houver disco de dados também;
- *(opcional)* Desabilitar temporariamente antivírus e firewall dentro do S.O.

> A Microsoft também recomenda a execução da ferramenta de assessment antes do processo para validar compatibilidade e possíveis impedimentos.
{: .prompt-info }

---

## Mão na massa!

#### Passo 1

A primeira tarefa é confirmar se a sua origem e o seu destino fazem parte de um caminho de upgrade suportado.

Na prática, não é porque a VM está no Azure que qualquer salto de versão será permitido. O ideal é sempre consultar a matriz oficial antes de começar, principalmente se você estiver trabalhando com versões mais antigas ou planejando um salto maior.

Hoje, para cenários não clusterizados, o Windows Server 2025 aceita upgrade direto a partir do **Windows Server 2012 R2 e versões posteriores**. Já para outras versões, a recomendação continua sendo validar a matriz oficial antes da execução.

![winserver-upgrade](assets/img/005/002-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

***Opcional***
<br>
A Microsoft sugere que executemos **[Ferramenta de avaliação de atualização do sistema operacional Windows da VM do Azure](https://learn.microsoft.com/pt-br/troubleshoot/azure/virtual-machines/windows/windows-vm-osupgradeassessment-tool)** onde valida se o Sistema Operacional possui compatibilidade com o modelo de upgrade in-place. É uma ferramenta bem simples e de fácil utilização, onde você irá realizar o download da ferramenta diretamente do [repositório oficial no Github](https://github.com/Azure/azure-support-scripts/blob/master/RunCommand/Windows/Windows_OSUpgrade_Assessment_Validation) para a VM que será realizada essa atualização e executá-lo.

Caso ocorra alguma falha, ele retornará uma imagem parecida com essa:
![winserver-upgrade](assets/img/005/003-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

> Em ambiente real, esse é o tipo de etapa que evita retrabalho antes mesmo da janela de manutenção começar.
{: .prompt-tip }

---

#### Passo 2

Como todo administrador, você precisa pensar sempre na proteção e restauração em caso de algum problema durante o processo de atualização. Nessa etapa iremos realizar um snapshot do disco para (caso seja necessário) restaurá-lo em seguida.

> Caso você tenha sido designado para realizar um upgrade de mais de **5 / 10 / 50** VMs, nós temos um artigo que lhe auxiliará a realizar esses snapshots de forma automatizada! [Criando Snapshots de VMs no Azure com TAGs](https://blog.ruizsolutions.online/posts/criando-snapshot-de-vms-atraves-de-tags/). 
{: .prompt-info }
<br>
Essa etapa é o que vai te salvar caso o upgrade falhe no meio do processo ou o sistema não volte de forma saudável.

> Upgrade in-place sem snapshot é aposta. E ambiente corporativo não combina com aposta. 
{: .prompt-danger }
<br>
1 - No portal do Azure, acesse o Disco da VM em questão em Disks > OS Disk:
![winserver-upgrade](assets/img/005/004-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>
![winserver-upgrade](assets/img/005/005-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

2 - Já no disco, selecione **"Create Snapshot"**, insira um nome fácil de identificar e seguindo as melhores práticas de nomemclatura sugeridas pelo CAF. Mude o tipo de snapshot para FULL e selecione o tipo de storage para Standard HDD (locally-redundant storage):

![winserver-upgrade](assets/img/005/006-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>
![winserver-upgrade](assets/img/005/007-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

E então siga com as demais configurações padrões até que seja gerado um snapshot do disco do Sistema Operacional:

![winserver-upgrade](assets/img/005/008-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

---

#### Passo 3

No Azure, o processo oficial para Windows Server utiliza uma mídia de upgrade em formato de **Managed Disk**, criada a partir de uma imagem especial do Marketplace.

Essa imagem utiliza como base:

- **Publisher:** `MicrosoftWindowsServer`
- **Offer:** `WindowsServerUpgrade`
- **SKU:** varia conforme a versão de destino, como por exemplo `server2025Upgrade`, `server2022Upgrade`, `server2019Upgrade`, `server2016Upgrade` ou `server2012Upgrade`

Abaixo está um exemplo de script em PowerShell para criar esse disco de mídia de upgrade.

1. Abra o **Cloud Shell** no portal do Azure (Para melhor visualização sugerimos que após o carregamento do Cloud Shell você clique em NOVA SESSÃO, assim ele abrirá uma nova aba para facilitar o manuseio);
2. Ajuste os parâmetros conforme a sua necessidade;
3. Execute o código abaixo e aguarde o fim da criação do disco.

```powershell
# Resource group of the source VM
$resourceGroup = "WindowsServerUpgrades"

# Location of the source VM
$location = "WestUS2"

# Zone of the source VM, if any
$zone = ""

# Disk name for the that will be created
$diskName = "WindowsServer2025UpgradeDisk"

# Target version for the upgrade - must be one of these five strings: server2025Upgrade, server2022Upgrade, server2019Upgrade, server2016Upgrade or server2012Upgrade
$sku = "server2025Upgrade"

# Common parameters

$publisher = "MicrosoftWindowsServer"
$offer = "WindowsServerUpgrade"
$managedDiskSKU = "Standard_LRS"

#
# Get the latest version of the special (hidden) VM Image from the Azure Marketplace

$versions = Get-AzVMImage -PublisherName $publisher -Location $location -Offer $offer -Skus $sku | sort-object -Descending {[version] $_.Version	}
$latestString = $versions[0].Version

# Get the special (hidden) VM Image from the Azure Marketplace by version - the image is used to create a disk to upgrade to the new version

$image = Get-AzVMImage -Location $location -PublisherName $publisher -Offer $offer -Skus $sku -Version $latestString

#
# Create Resource Group if it doesn't exist
#

if (-not (Get-AzResourceGroup -Name $resourceGroup -ErrorAction SilentlyContinue)) {
    New-AzResourceGroup -Name $resourceGroup -Location $location    
}

#
# Create Managed Disk from LUN 0
#

if ($zone){
    $diskConfig = New-AzDiskConfig -SkuName $managedDiskSKU -CreateOption FromImage -Zone $zone -Location $location
} else {
    $diskConfig = New-AzDiskConfig -SkuName $managedDiskSKU -CreateOption FromImage -Location $location
}

Set-AzDiskImageReference -Disk $diskConfig -Id $image.Id -Lun 0

New-AzDisk -ResourceGroupName $resourceGroup -DiskName $diskName -Disk $diskConfig
```

Ficará assim:
![winserver-upgrade](assets/img/005/009-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>
![winserver-upgrade](assets/img/005/010-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

> OBS 1 - Caso você não saiba o nome corretamente da região do seu recurso, acesse essa lista -> [List of All Regions on Azure](https://learn.microsoft.com/en-us/azure/reliability/regions-list?tabs=all)
{: .prompt-info }
> OBS 2 - Esse disco deve ser criado na mesma região da VM e, se a VM estiver em zona, preferencialmente na mesma zona também. 
{: .prompt-info }
> OBS 3 - O disco que foi provisionado possui um espaço de exatamente 10GB apenas para essa função. 
{: .prompt-info }


#### Passo 4

Com o disco criado, vamos anexar esse disco de upgrade na VM.

1 - No portal do Azure, navegue até a sua VM > clique em Disks > clique em Attach existing disks > Selecione o disco de upgrade criado anteriormente > e salve a configuração.

![winserver-upgrade](assets/img/005/011-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

> Esse disco precisa estar na mesma região da VM e, se a VM estiver em zona, na mesma zona também. 
{: .prompt-info }

---

#### Passo 5

Agora sim vem a cereja do bolo, com a VM em execução, conecte-se via RDP ou Azure Bastion e identifique se o disco foi montado corretamente na VM.

1 - No botão do Windows, clique com o botão direito e selecione Gerenciamento de Disco (Ou Disk Management):
![winserver-upgrade](assets/img/005/012-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>
![winserver-upgrade](assets/img/005/013-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

Se está tudo OK, podemo seguir com a atualização.

2 - Abra o PowerShell com privilégios de administrador e então navegue até a unidade onde o disco de upgrade foi montado:
![winserver-upgrade](assets/img/005/014-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

3 - Nesse ponto vamos executar a atualização do Windows, para isso você precisará validar qual a versão do Windows a ser executada.
Para Windows Server 2016, 2019, 2022 ou 2025, você pode utilizar o seguinte comando:

```powershell
.\setup.exe /auto upgrade /dynamicupdate disable /eula accept
```

Esse comando ajuda a automatizar o processo e evita travas por aceite manual do contrato de licença.

Ou, se você estiver lidando especificamente com um alvo Windows Server 2012, o fluxo é mais manual e o setup é iniciado apenas com:

```powershell
.\setup.exe
```

![winserver-upgrade](assets/img/005/015-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>
![winserver-upgrade](assets/img/005/016-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

Depois disso, irá aparecer as opções de seleção para qual sistema operacional será realizado o upgrade:

![winserver-upgrade](assets/img/005/017-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

> Selecione com atenção a imagem correta de destino conforme a versão atual e a matriz de upgrade suportada. 
{: .prompt-warning }
<br>
Em seguida o upgrade in-place estará sendo executado, nesse ponto é só aguardar.

![winserver-upgrade](assets/img/005/018-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

---

#### Passo 6

Durante o processo, a VM vai reiniciar e sua sessão RDP será desconectada automaticamente e isso é esperado.

Nesse momento, você pode acompanhar o progresso por meio da funcionalidade de screenshot / boot diagnostics no portal do Azure, o que é muito útil quando a VM ainda não voltou para acesso remoto.

![winserver-upgrade](assets/img/005/019-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

> Por se tratar de um snapshot (print de tela), ele não atualizará a imagem sozinho, você pode clicar no botão Refresh/Atualizar logo acima para acompanhar todo o processo de atualização:
{: .prompt-tip }
<br>
![winserver-upgrade](assets/img/005/020-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

Quando a atualização de fato estiver finalizada, aparecerá uma tela semelhante a essa abaixo, o que quer dizer que podemos ir para a finalização de todo processo:
![winserver-upgrade](assets/img/005/021-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

---

#### Passo 7

Concluído o upgrade, valide cuidadosamente o ambiente:
* Verifique a versão do Windows Server;
![winserver-upgrade](assets/img/005/022-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>
![winserver-upgrade](assets/img/005/023-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>
* Valide serviços e aplicações instaladas;
* Confirme a conectividade de rede;
* Revise roles e features;
* Verifique o status de ativação/licenciamento;
* Confirme a integridade geral da VM.
* Depois disso, faça a limpeza dos artefatos temporários:
* * Remova o disco de mídia de upgrade;
* * Reabilite antivírus, antispyware e firewall, caso tenham sido desabilitados;
* * Mantenha os snapshots por uma janela segura e remova-os quando não forem mais necessários.

> Utilize a limpeza de Disco e selecione todos os itens disponíveis incluindo as instalações anteriores para liberar espaço e não levar nenhum lixo a frente. 
{: .prompt-warning }

![winserver-upgrade](assets/img/005/024-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>
![winserver-upgrade](assets/img/005/025-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

---

Outro detalhe também que você irá notar no Portal do Azure é que a versão informada está devidamente atualizada:

**Versão anterior:**
![winserver-upgrade](assets/img/005/026-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

**Versão Atual:**
![winserver-upgrade](assets/img/005/027-windows-server-upgrade-azure.png){: .shadow .rounded-10}
<br>

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

E chegamos ao fim de mais um artigo em nosso blog!

A ideia aqui foi te mostrar que, **sim é possível realizar um upgrade in-place de Windows Server em uma VM do Azure de forma suportada e organizada**, desde que você respeite o caminho de upgrade, proteja rollback e entenda os impactos operacionais no Azure.

No próximo artigo irei trazer como realizar esta mesma configuração, porém em uma VM Client, ou seja, iremos atualizar juntos uma VM com Windows 10 para Windows 11!

Até a próxima!