---
#layout: post
title: "Removendo Snapshots de forma automatizada"
date: 2025-08-31 12:00:00
categories: [Azure]
tags: [azure, tag, environment, snapshot, resources]
slug: 'removendo-snapshots-de-forma-automatizada'
image:
  path: assets/img/004/001-removendo-snapshot-vms.png
---

Fala pessoALL! Daremos continuidade ao assunto que abordamos anteriormente, onde fizemos a criação em massa de forma simples e eficiente usando um script pronto onde podemos facilmente customiza-lo!

Entretanto se criamos diversos snapshots, cada um com as TAGs necessárias, como faremos a deleção desses recursos? Em tempos passados faríamos na **MÃO**, correto?

Porém, fazer na mão 5 ou 10 é até aceitável… Agora fazer 50, 100 ou 200 na mão? Certamente seu superior irá colocar o ChatGPT no seu lugar!

**Neste artigo mostrarei em como excluir os Snapshots que foram criados anteriormente com contendo algumas TAGs previamente preenchidas, e assim evitarmos quaisquer problemas de exclusão incorreta.**

---

### Pre-requisitos:
- Possuir permissão de no mínimo Contributor da Subscription
- Instalar o [PowerShell (mínimo 7.4.2)](https://github.com/powershell/powershell/releases)
- Instalar o módulo de [Az](https://learn.microsoft.com/en-us/powershell/azure/install-azps-windows?view=azps-14.3.0&viewFallbackFrom=azps-13.4.0&tabs=powershell&pivots=windows-psgallery)

---

## Mão na massa!

#### Passo 1

**Acesse o meu repositório e baixe o arquivo [removesnapshot.ps1](https://github.com/lfrleite/Ruiz-Online/blob/main/Removendo%20Snapshots/removesnapshot.ps1). Ou se preferir, copie o [código](https://github.com/lfrleite/Ruiz-Online/tree/main/Removendo%20Snapshots) e crie um arquivo chamado *removesnapshot.ps1***
<br>

1 - Crie (Ou reutilize) uma pasta chamada **Temp** na raiz da unidade C:\ e extraia (ou copie o arquivo que criou copiando o código) o arquivo neste local:

![remove-snap-config](/assets/img/004/002-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

2 - O arquivo **‘removesnapshot.ps1‘** encontra-se devidamente preenchido contendo as informações necessárias para remover os snapshots que foram criadas com as TAGs informadas anteriormente:

![remove-snap-config](/assets/img/004/003-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

> Nosso objetivo será excluir os snapshots contendo as tags **“Chamado“**, **“Solicitante”** e **“Excluir em“**, para isso validamos no Portal do Azure o nome corretamente para utiliza-lo depois no Powershell:
{: .prompt-info } 

![remove-snap-config](/assets/img/004/004-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

---

#### Passo 2

1 - Abra o arquivo o PowerShell e navegue até a pasta **C:\Temp**:

![remove-snap-config](/assets/img/004/005-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

2 - Nesse momento, execute o removesnapshot.ps1 incluindo os parâmetros **TenantId | Chamado | Excluir | Solicitante**:
<br>

```powershell
.\removesnapshot.ps1 -TenantId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -Chamado "NumDoChamado" -Excluir "DataParaExclusão" -Solicitante "NomeDoSolicitante"
```

![remove-snap-config](/assets/img/004/006-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

3 - Com todas as informações devidamente preenchidas é só clicar em **ENTER** no teclado, aparecerá um ‘pop-up’ solicitando uma conta que tenha acesso ao Tenant indicado anteriormente:

![remove-snap-config](/assets/img/004/007-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

4 - Autenticação realizada com sucesso, o PowerShell irá lhe retornar com as informações dos snapshots que foram listados com as tags **“Chamado“**, **“Solicitante”** e **“Excluir em”** onde você poderá fazer uma dupla checagem antes de seguir adiante:

![remove-snap-config](/assets/img/004/008-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

> Validação realizada com sucesso, basta digitar **“S”** e apertar **ENTER** novamente.
{: .prompt-info } 

---

#### Passo 3

Agora podemos acompanhar a deleção sendo feita de **TODOS** os snapshots de forma simples e rápida!

1 - A depender da quantidade de snapshots realizados anteriormente, pode durar entre 5 a 15 minutos:

![remove-snap-config](/assets/img/004/009-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

2 - Ao fim da execução não irá retornar uma mensagem, contudo poderá validar diretamente no portal do Azure se todos foram excluídos com sucesso:

**ANTES**
![remove-snap-config](/assets/img/004/010-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

**DEPOIS**
![remove-snap-config](/assets/img/004/011-removendo-snapshot-vms.png){: .shadow .rounded-10} 
<br>

---

### Checklist
- [x] Passo 1 - Fazer o download dos arquivos necessários e inserir as informações para o removesnapshot.ps1
- [x] Passo 2 - Executar o arquivo removesnapshot.ps1 com os parâmetros TenantId, Chamado, Excluir e Solicitante
- [x] Passo 3 - Acompanhar e validar a remoção de todos os snapshots

---

## Artigos

| Nome                                                        | Link                                                                                                                               |
| :----------------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------------:|
| Instalar o PowerShell                                       | <https://github.com/powershell/powershell/releases>                                                                                |
| Instalar o módulo Az                                        | <https://learn.microsoft.com/pt-br/powershell/azure/install-azps-windows?view=azps-12.0.0&tabs=powershell&pivots=windows-psgallery>|

---

## The End!

Este é o fim de mais um artigo em nosso blog! Novamente espero que vocês tenham curtido e seja bem útil a toda comunidade de TI! Espero que tenham curtido este artigo... Até a próxima!