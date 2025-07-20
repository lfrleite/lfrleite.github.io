---
#layout: post
title: "Inventariando qualquer ambiente do Azure com o Azure Resource Inventory (ARI)"
date: 2025-07-20 12:00:00
categories: [Azure]
tags: [azure, tag, environment, inventory, resources]
slug: 'inventariando-qualquer-ambiente-do-azure-com-ari'
image:
  path: assets/img/002/001-azure-inventory.png
---

Fala PessoALL! Um dos primeiros passos como um arquiteto Azure é identificar e mensurar todos os recursos de um cliente em um ambiente já existente. Mas como fazer isso de forma prática, simples e o melhor: DE GRAÇA? - Usando o Azure Resource Inventory ou ARI.

Contudo muitos ainda fazem a mesma pergunta: “Para que e por que usar o ARI?” – Simples! O ARI tem como objetivo gerar um inventário de forma automatizada a grande maioria dos recursos alocados no Azure, gerando um arquivo .xlsx (Excel) e diagrama (draw.io).

**Neste artigo, abordaremos como inventariar todo ambiente do Azure de forma prática, utilizando o menor privilégio possível e entregar a melhor experiência possível para o nosso cliente.**

---

### Pre-requisitos:
- Possuir permissão de no mínimo Leitor/Reader da Subscription
- Instalar o [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- Instalar o módulo de [PowerShell ImportExcel](https://github.com/dfinke/ImportExcel)
- Instalar o aplicativo [Draw.io](https://apps.microsoft.com/detail/9mvvszk43qqw?hl=pt-br&gl=br)
- Utilizar o PowerShell 7.0+

---

## Mão na massa!

> **“Houve uma atualização no uso do Azure Resource Inventory, passando a utilizar um módulo de PowerShell ao invés de usar o arquivo .ps1 que usamos neste artigo”**
{: .prompt-warning } 
<br>

> **Para melhores resultados e manter as orientações abaixo neste artigo (onde testamos e validamos o funcionamento), utilizaremos a release na versão 3.1.16.**
{: .prompt-info } 
<br>

#### Passo 1

1 - Acesse o link do [ARI](https://github.com/microsoft/ARI/releases/tag/3.1.16) para realizar o download do arquivo .ZIP:

![ari-deploy](/assets/img/002/002-azure-inventory.png){: .shadow .rounded-10} 
<br>


2 - Na raiz da unidade **‘C:\’** crie uma pasta pasta chamada *‘AzureResourceInventory‘* e extraia o arquivo **‘ARI-main.ZIP‘** nessa pasta, pois dessa forma ficarão centralizados as informações tendo em vista que o resultado do inventário salvará os arquivos nessa pasta, conforme informado na página oficial deste recurso:

![ari-deploy](/assets/img/002/003-azure-inventory.png){: .shadow .rounded-10} 
<br>

![ari-deploy](/assets/img/002/004-azure-inventory.png){: .shadow .rounded-10} 
<br>

---

#### Passo 2

1 - Execute o Windows PowerShell (**NÃO** precisa executar como ~~administrador~~) e navegue até o caminho onde está localizado o arquivo de execução do PowerShell chamado ‘AzureResourceInventory.ps1‘:

**C:\AzureResourceInventory\ARI-main**

![ari-deploy](/assets/img/002/005-azure-inventory.png){: .shadow .rounded-10} 
<br>

![ari-deploy](/assets/img/002/006-azure-inventory.png){: .shadow .rounded-10} 
<br>

---

#### Passo 3

1 - Para o sucesso da execução do script, se faz necessário executar o **‘.\AzureResourceInventory.ps1‘* incluindo os parâmetros:
**-TenantID "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -SubscriptionID "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -IncludeTags -Diagram -QuotaUsage -SkipAdvisory**

```powershell
.\AzureResourceInventory.ps1 -TenantID "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -SubscriptionID "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -IncludeTags -Diagram -QuotaUsage -SkipAdvisory
```

![ari-deploy](/assets/img/002/007-azure-inventory.png){: .shadow .rounded-10} 
<br>

> **(OPCIONAL o uso do parâmetro -SecurityCenter)**
{: .prompt-info } 
<br>

---

> Caso aconteça alguma falha referente ao “ExecutionPolicy” do PowerShell, valide utilizando o comando **Get-ExecutionPolicy -List**
{: .prompt-danger } 
<br>

![ari-deploy](/assets/img/002/008-azure-inventory.png){: .shadow .rounded-10} 
<br>

Para resolvermos este problema precisaremos executar um novo comando, passando o parâmetro Bypass no escopo CurrentUser:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Bypass
```

Após executar o script o retorno aparecerá desta forma abaixo:

![ari-deploy](/assets/img/002/009-azure-inventory.png){: .shadow .rounded-10} 
<br>

> Obs.: Caso tenha alguma atualização de módulo ele fará automaticamente
{: .prompt-info } 
<br>

2 - Em seguida será direcionado ao seu navegador solicitando que autentique a sua conta, utilize a conta que tenha acesso ao **Tenant** e a **Subscription** informada anteriormente:

![ari-deploy](/assets/img/002/010-azure-inventory.png){: .shadow .rounded-10} 
<br>

E pronto! Nesse momento você pode retornar a janela do PowerShell e acompanhar o processo de inventário ser concluído:

![ari-deploy](/assets/img/002/011-azure-inventory.png){: .shadow .rounded-10} 
<br>

---

#### Passo 4

1 - Ao fim da execução do script o PowerShell irá apresentar um resumo de tempo, total de recursos, total de avisos/orientações, o nome do dashboard exportado em Excel e o arquivo gráfico para ser aberto através do aplicativo Draw.io:

![ari-results](/assets/img/002/012-azure-inventory.png){: .shadow .rounded-10} 
<br>

![ari-results](/assets/img/002/013-azure-inventory.png){: .shadow .rounded-10} 
<br>

2 - Verifique os dados coletados na planilha .xlsx – *‘AzureResourceInventory_Report_yyyy-MM-dd_HH_mm.xlsx‘*:

![ari-results](/assets/img/002/014-azure-inventory.png){: .shadow .rounded-10} 
<br>

> **Você pode validar que ele além de lhe trazer um dashboard completo com todos os recursos, ainda separa seus recursos por páginas dentro da mesma planilha.**
{: .prompt-info } 

3 - Com a planilha em mãos você já poderá analisar alguns itens que ficaram **‘órfãos’** de seus recursos e continuarão sendo cobrados sem utilização, dependendo do SKU que os recursos estão pode gerar um excelente **‘saving’** nos custos mensais:

![ari-results](/assets/img/002/015-azure-inventory.png){: .shadow .rounded-10} 
<br>

![ari-results](/assets/img/002/016-azure-inventory.png){: .shadow .rounded-10} 
<br>

---

#### Passo 5

1 - Para abrir o gráfico gerado, primeiro deve abrir o aplicativo Draw.io, criar um novo diagrama e então importar o arquivo gerado com nome **‘AzureResourceInventory_Diagram_yyyy-MM-dd_HH_mm.xml‘**:

![ari-results](/assets/img/002/017-azure-inventory.png){: .shadow .rounded-10} 
<br>

![ari-results](/assets/img/002/018-azure-inventory.png){: .shadow .rounded-10} 
<br>

![ari-results](/assets/img/002/019-azure-inventory.png){: .shadow .rounded-10} 
<br>

> Após essa importação o diagrama estará disponível com 3 ou mais páginas, subdividindo as informações entre elas:

![ari-results](/assets/img/002/020-azure-inventory.png){: .shadow .rounded-10} 
<br>

> **Como o meu tenant não possui VPN ele informou apenas o “Cloud Only Environment”, caso tivesse uma VPN ele apareceria “On Premises Environment”.**
{: .prompt-info } 

![ari-results](/assets/img/002/021-azure-inventory.png){: .shadow .rounded-10} 
<br>

> **Sempre lembrando que por se tratar de um recurso gratuito ele é limitado em sua disposição e layout, mas a vantagem é que você é livre para manuseá-lo da sua maneira, acrescentando ou removendo itens a sua escolha!**
{: .prompt-warning } 

---

### Checklist
- [x] Passo 1 - Download do ARI e ajustar o armazenamento padrão
- [x] Passo 2 - Executar o PowerShell sem privilégios de Administrador e acessar o caminho informado acima
- [x] Passo 3 - Executar o AzureResourceInventory.ps1 com os parâmetros TenantdId e SubscriptionId
- [x] Passo 4 - Obtendo resultados de todo inventário no ambiente do Azure
- [x] Passo 5 - Importar o arquivo .xml para o Draw.io e obter maiores informações de forma gráfica

---

## Inspiração

**Esse artigo foi inspirado no vídeo do Raphael Andrade**:

<iframe width="800" height="512" src="https://www.youtube.com/embed/vKk7E26b1e8" title="Como inventariar seu ambiente Azure" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## Artigos

| Nome                                                        | Link                                                                                                     |
| :----------------------------------------------------------:|:--------------------------------------------------------------------------------------------------------:|
| ARI (Azure Resource Inventory)                              | <https://github.com/microsoft/ARI>                                                                       |
| Como instalar a CLI do Azure                                | <https://learn.microsoft.com/pt-br/cli/azure/install-azure-cli>                                          |
| Extensões ‘az’                                              | <https://learn.microsoft.com/pt-br/cli/azure/extension?view=azure-cli-latest#az-extension-list-available>|
| Módulo de PowerShell ImportExcel                            | <https://github.com/dfinke/ImportExcel>                                                                  |
| Download do Draw.io                                         | <https://apps.microsoft.com/detail/9mvvszk43qqw?hl=pt-br&gl=br>                                          |

---

## The End!

Este é o fim de mais um artigo em nosso blog! Espero que vocês possam usufruir dessa potente ferramenta e que esse artigo tenha facilitado de alguma forma o seu uso. Até a próxima!