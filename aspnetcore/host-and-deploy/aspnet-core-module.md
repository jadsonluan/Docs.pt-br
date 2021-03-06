---
title: Referência de configuração do Módulo do ASP.NET Core
author: guardrex
description: Saiba como configurar o módulo do ASP.NET Core para hospedar aplicativos do ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 02/15/2018
uid: host-and-deploy/aspnet-core-module
ms.openlocfilehash: bf7a60b67b1ea78bb346e6dd5eeef38b54bfdbe4
ms.sourcegitcommit: b2723654af4969a24545f09ebe32004cb5e84a96
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/18/2018
ms.locfileid: "46010943"
---
# <a name="aspnet-core-module-configuration-reference"></a>Referência de configuração do Módulo do ASP.NET Core

Por [Luke Latham](https://github.com/guardrex), [Rick Anderson](https://twitter.com/RickAndMSFT) e [Sourabh Shirhatti](https://twitter.com/sshirhatti)

Este documento fornece instruções sobre como configurar o Módulo do ASP.NET Core para hospedar aplicativos do ASP.NET Core. Para obter uma introdução ao Módulo do ASP.NET Core e instruções de instalação, veja a [visão geral do Módulo do ASP.NET Core](xref:fundamentals/servers/aspnet-core-module).

## <a name="configuration-with-webconfig"></a>Configuração com web.config

O Módulo do ASP.NET Core está configurado com a seção `aspNetCore` do nó `system.webServer` no arquivo *web.config* do site.

O seguinte arquivo *web.config* é publicado para uma [implantação dependente de estrutura](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd) e configura o Módulo do ASP.NET Core para manipular solicitações de site:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="dotnet" 
                arguments=".\MyApp.dll" 
                stdoutLogEnabled="false" 
                stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

O seguinte *web.config* é publicado para uma [implantação autossuficiente](/dotnet/articles/core/deploying/#self-contained-deployments-scd):

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath=".\MyApp.exe" 
                stdoutLogEnabled="false" 
                stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

Quando um aplicativo é implantado no [Serviço de Aplicativo do Azure](https://azure.microsoft.com/services/app-service/), o caminho `stdoutLogFile` é definido para `\\?\%home%\LogFiles\stdout`. O caminho salva logs de stdout para a pasta *LogFiles*, que é um local criado automaticamente pelo serviço.

Confira [Configuração de subaplicativos](xref:host-and-deploy/iis/index#sub-application-configuration) para uma nota importante relativa à configuração de arquivos *web.config* em subaplicativos.

### <a name="attributes-of-the-aspnetcore-element"></a>Atributos do elemento aspNetCore

::: moniker range="<= aspnetcore-2.0"

| Atributo | Descrição | Padrão |
| --------- | ----------- | :-----: |
| `arguments` | <p>Atributo de cadeia de caracteres opcional.</p><p>Argumentos para o executável especificado em **processPath**.</p>| |
| `disableStartUpErrorPage` | verdadeiro ou falso.</p><p>Se for true, a página **502.5 – Falha do Processo** será suprimida e a página de código de status 502, configurada no *web.config*, terá precedência.</p> | `false` |
| `forwardWindowsAuthToken` | verdadeiro ou falso.</p><p>Se for true, o token será encaminhado para o processo filho escutando em %ASPNETCORE_PORT% como um cabeçalho 'MS-ASPNETCORE-WINAUTHTOKEN' por solicitação. É responsabilidade desse processo chamar CloseHandle nesse token por solicitação.</p> | `true` |
| `processPath` | <p>Atributo de cadeia de caracteres obrigatório.</p><p>Caminho para o executável que inicia um processo que escuta solicitações HTTP. Caminhos relativos são compatíveis. Se o caminho começa com `.`, o caminho é considerado relativo à raiz do site.</p> | |
| `rapidFailsPerMinute` | <p>Atributo inteiro opcional.</p><p>Especifica o número de vezes que o processo especificado em **processPath** pode falhar por minuto. Se esse limite for excedido, o módulo interromperá a inicialização do processo pelo restante do minuto.</p> | `10` |
| `requestTimeout` | <p>Atributo de intervalo de tempo opcional.</p><p>Especifica a duração para a qual o Módulo do ASP.NET Core aguarda uma resposta do processo que escuta em %ASPNETCORE_PORT%.</p><p>Em versões do Módulo ASP.NET Core que acompanham a versão do ASP.NET Core 2.0 ou anterior, o `requestTimeout` deve ser especificado somente em minutos inteiros, caso contrário, ele assume o valor padrão de 2 minutos.</p> | `00:02:00` |
| `shutdownTimeLimit` | <p>Atributo inteiro opcional.</p><p>Duração em segundos que o módulo espera para o executável desligar normalmente quando o arquivo *app_offline.htm* é detectado.</p> | `10` |
| `startupTimeLimit` | <p>Atributo inteiro opcional.</p><p>Duração em segundos que o módulo espera para o arquivo executável iniciar um processo escutando na porta. Se esse tempo limite é excedido, o módulo encerra o processo. O módulo tentará reiniciar o processo quando ele receber uma nova solicitação e continuará a tentar reiniciar o processo em solicitações subsequentes de entrada, a menos que o aplicativo falhe em iniciar um número de vezes igual a **rapidFailsPerMinute** no último minuto sem interrupção.</p> | `120` |
| `stdoutLogEnabled` | <p>Atributo booliano opcional.</p><p>Se for true, **stdout** e **stderr** para o processo especificado em **processPath** serão redirecionados para o arquivo especificado em **stdoutLogFile**.</p> | `false` |
| `stdoutLogFile` | <p>Atributo de cadeia de caracteres opcional.</p><p>Especifica o caminho relativo ou absoluto para o qual **stdout** e **stderr** do processo especificado em **processPath** são registrados em log. Os caminhos relativos são relativos à raiz do site. Qualquer caminho começando com `.` é relativo à raiz do site e todos os outros caminhos são tratados como caminhos absolutos. As pastas fornecidas no caminho devem existir para que o módulo crie o arquivo de log. Usando delimitadores de sublinhado, um carimbo de data/hora, uma ID de processo e a extensão de arquivo (*.log*) são adicionados ao último segmento do caminho **stdoutLogFile**. Se `.\logs\stdout` é fornecido como um valor, um log de exemplo stdout é salvo como *stdout_20180205194132_1934.log* na pasta *logs* quando salvos em 5/2/2018, às 19:41:32, com uma ID de processo de 1934.</p> | `aspnetcore-stdout` |

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

| Atributo | Descrição | Padrão |
| --------- | ----------- | :-----: |
| `arguments` | <p>Atributo de cadeia de caracteres opcional.</p><p>Argumentos para o executável especificado em **processPath**.</p>| |
| `disableStartUpErrorPage` | verdadeiro ou falso.</p><p>Se for true, a página **502.5 – Falha do Processo** será suprimida e a página de código de status 502, configurada no *web.config*, terá precedência.</p> | `false` |
| `forwardWindowsAuthToken` | verdadeiro ou falso.</p><p>Se for true, o token será encaminhado para o processo filho escutando em %ASPNETCORE_PORT% como um cabeçalho 'MS-ASPNETCORE-WINAUTHTOKEN' por solicitação. É responsabilidade desse processo chamar CloseHandle nesse token por solicitação.</p> | `true` |
| `processPath` | <p>Atributo de cadeia de caracteres obrigatório.</p><p>Caminho para o executável que inicia um processo que escuta solicitações HTTP. Caminhos relativos são compatíveis. Se o caminho começa com `.`, o caminho é considerado relativo à raiz do site.</p> | |
| `rapidFailsPerMinute` | <p>Atributo inteiro opcional.</p><p>Especifica o número de vezes que o processo especificado em **processPath** pode falhar por minuto. Se esse limite for excedido, o módulo interromperá a inicialização do processo pelo restante do minuto.</p> | `10` |
| `requestTimeout` | <p>Atributo de intervalo de tempo opcional.</p><p>Especifica a duração para a qual o Módulo do ASP.NET Core aguarda uma resposta do processo que escuta em %ASPNETCORE_PORT%.</p><p>Em versões do Módulo do ASP.NET Core que acompanham a versão do ASP.NET Core 2.1 ou posterior, o `requestTimeout` é especificado em horas, minutos e segundos.</p> | `00:02:00` |
| `shutdownTimeLimit` | <p>Atributo inteiro opcional.</p><p>Duração em segundos que o módulo espera para o executável desligar normalmente quando o arquivo *app_offline.htm* é detectado.</p> | `10` |
| `startupTimeLimit` | <p>Atributo inteiro opcional.</p><p>Duração em segundos que o módulo espera para o arquivo executável iniciar um processo escutando na porta. Se esse tempo limite é excedido, o módulo encerra o processo. O módulo tentará reiniciar o processo quando ele receber uma nova solicitação e continuará a tentar reiniciar o processo em solicitações subsequentes de entrada, a menos que o aplicativo falhe em iniciar um número de vezes igual a **rapidFailsPerMinute** no último minuto sem interrupção.</p> | `120` |
| `stdoutLogEnabled` | <p>Atributo booliano opcional.</p><p>Se for true, **stdout** e **stderr** para o processo especificado em **processPath** serão redirecionados para o arquivo especificado em **stdoutLogFile**.</p> | `false` |
| `stdoutLogFile` | <p>Atributo de cadeia de caracteres opcional.</p><p>Especifica o caminho relativo ou absoluto para o qual **stdout** e **stderr** do processo especificado em **processPath** são registrados em log. Os caminhos relativos são relativos à raiz do site. Qualquer caminho começando com `.` é relativo à raiz do site e todos os outros caminhos são tratados como caminhos absolutos. As pastas fornecidas no caminho devem existir para que o módulo crie o arquivo de log. Usando delimitadores de sublinhado, um carimbo de data/hora, uma ID de processo e a extensão de arquivo (*.log*) são adicionados ao último segmento do caminho **stdoutLogFile**. Se `.\logs\stdout` é fornecido como um valor, um log de exemplo stdout é salvo como *stdout_20180205194132_1934.log* na pasta *logs* quando salvos em 5/2/2018, às 19:41:32, com uma ID de processo de 1934.</p> | `aspnetcore-stdout` |

::: moniker-end

### <a name="setting-environment-variables"></a>Definindo variáveis de ambiente

Variáveis de ambiente podem ser especificadas para o processo no atributo `processPath`. Especificar uma variável de ambiente com o elemento filho `environmentVariable` de um elemento de coleção `environmentVariables`. Variáveis de ambiente definidas nesta seção têm precedência sobre variáveis de ambiente do sistema.

O exemplo a seguir define duas variáveis de ambiente. `ASPNETCORE_ENVIRONMENT` configura o ambiente do aplicativo para `Development`. Um desenvolvedor pode definir esse valor temporariamente no arquivo *web.config* para forçar o carregamento da [Página de Exceções do Desenvolvedor](xref:fundamentals/error-handling) ao depurar uma exceção de aplicativo. `CONFIG_DIR` é um exemplo de uma variável de ambiente definida pelo usuário, em que o desenvolvedor escreveu código que lê o valor de inicialização para formar um caminho no qual carregar o arquivo de configuração do aplicativo.

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile="\\?\%home%\LogFiles\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!WARNING]
> Defina a variável de ambiente `ASPNETCORE_ENVIRONMENT` para `Development` em servidores de preparo e de teste que não estão acessíveis a redes não confiáveis, tais como a Internet.

## <a name="appofflinehtm"></a>app_offline.htm

Se um arquivo com o nome *app_offline.htm* é detectado no diretório raiz de um aplicativo, o Módulo do ASP.NET Core tenta desligar normalmente o aplicativo e parar o processamento de solicitações de entrada. Se o aplicativo ainda está em execução após o número de segundos definido em `shutdownTimeLimit`, o Módulo do ASP.NET Core encerra o processo em execução.

Enquanto o arquivo *app_offline.htm* estiver presente, o Módulo do ASP.NET Core responderá às solicitações enviando o conteúdo do arquivo *app_offline.htm*. Quando o arquivo *app_offline.htm* é removido, a próxima solicitação inicia o aplicativo.

## <a name="start-up-error-page"></a>Página de erro de inicialização

Se o Módulo do ASP.NET Core falhar ao iniciar o processo de back-end ou se o processo de back-end iniciar mas falhar em escutar na porta configurada, uma página de código de status *502.5 – Falha no Processo* será exibida. Para omitir esta página e reverter para a página de código de status 502 padrão do IIS, use o atributo `disableStartUpErrorPage`. Para obter mais informações sobre como configurar mensagens de erro personalizadas, veja [Erros HTTP `<httpErrors>`](/iis/configuration/system.webServer/httpErrors/).

![Página de código de status 502.5 – Falha do Processo](aspnet-core-module/_static/ANCM-502_5.png)

## <a name="log-creation-and-redirection"></a>Criação de log e redirecionamento

O Módulo do ASP.NET Core redireciona as saídas de console stdout e stderr para o disco se os atributos `stdoutLogEnabled` e `stdoutLogFile` do elemento `aspNetCore` forem definidos. As pastas no caminho `stdoutLogFile` devem existir para que o módulo crie o arquivo de log. O pool de aplicativos deve ter acesso de gravação ao local em que os logs foram gravados (use `IIS AppPool\<app_pool_name>` para fornecer permissão de gravação).

Logs não sofrem rotação, a menos que ocorra a reciclagem/reinicialização do processo. É responsabilidade do hoster limitar o espaço em disco consumido pelos logs.

Usar o log de stdout é recomendado apenas para solucionar problemas de inicialização do aplicativo. Não use o log de stdout para fins gerais de registro em log do aplicativo. Para registro em log de rotina em um aplicativo ASP.NET Core, use uma biblioteca de registro em log que limita o tamanho do arquivo de log e realiza a rotação de logs. Para obter mais informações, veja [provedores de log de terceiros](xref:fundamentals/logging/index#third-party-logging-providers).

Uma extensão de arquivo e um carimbo de data/hora são adicionados automaticamente quando o arquivo de log é criado. O nome do arquivo de log é composto por meio do acréscimo do carimbo de data/hora, da ID do processo e da extensão de arquivo (*.log*) para o último segmento do caminho `stdoutLogFile` (normalmente *stdout*), delimitados por sublinhados. Se o caminho `stdoutLogFile` termina com *stdout*, um log para um aplicativo com um PID de 1934, criado em 5/2/2018 às 19:42:32, tem o nome de arquivo *stdout_20180205194132_1934.log*.

O elemento `aspNetCore` de exemplo a seguir configura o registro em log de stdout para um aplicativo hospedado no Serviço de Aplicativo do Azure. Um caminho local ou um caminho de compartilhamento de rede é aceitável para o registro em log local. Confirme se a identidade do usuário AppPool tem permissão para gravar no caminho fornecido.

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile="\\?\%home%\LogFiles\stdout">
</aspNetCore>
```

Veja [Configuração com web.config](#configuration-with-webconfig) para obter um exemplo do elemento `aspNetCore` no arquivo *web.config*.

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a>A configuração de proxy usa o protocolo HTTP e um token de emparelhamento

O proxy criado entre o Módulo do ASP.NET Core e o Kestrel usa o protocolo HTTP. O uso de HTTP é uma otimização de desempenho na qual o tráfego entre o módulo e o Kestrel ocorre em um endereço de loopback fora do adaptador de rede. Não há nenhum risco de interceptação do tráfego entre o módulo e o Kestrel em um local fora do servidor.

Um token de emparelhamento é usado para assegurar que as solicitações recebidas pelo Kestrel foram transmitidas por proxy pelo IIS e que não são provenientes de outra origem. O token de emparelhamento é criado e definido em uma variável de ambiente (`ASPNETCORE_TOKEN`) pelo módulo. O token de emparelhamento também é definido em um cabeçalho (`MSAspNetCoreToken`) em cada solicitação com proxy. O Middleware do IIS verifica cada solicitação recebida para confirmar se o valor de cabeçalho do token de emparelhamento corresponde ao valor da variável de ambiente. Se os valores do token forem incompatíveis, a solicitação será registrada em log e rejeitada. A variável de ambiente do token de emparelhamento e o tráfego entre o módulo e o Kestrel não são acessíveis em um local fora do servidor. Sem saber o valor do token de emparelhamento, um invasor não pode enviar solicitações que ignoram a verificação no Middleware do IIS.

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a>Módulo do ASP.NET Core com uma configuração do IIS compartilhada

O instalador do módulo do ASP.NET Core é executado com os privilégios da conta de **SISTEMA**. Já que a conta de sistema local não tem permissão para modificar o caminho do compartilhamento usado pela configuração compartilhada de IIS, o instalador experimenta um erro de acesso negado ao tentar definir as configurações de módulo em *applicationHost.config* no compartilhamento. Ao usar uma configuração compartilhada de IIS, siga estas etapas:

1. Desabilite a configuração compartilhada de IIS.
1. Execute o instalador.
1. Exportar o arquivo *applicationHost.config* atualizado para o compartilhamento.
1. Reabilite a Configuração Compartilhada do IIS.

## <a name="module-version-and-hosting-bundle-installer-logs"></a>Versão do módulo e logs do instalador do pacote de hospedagem

Para determinar a versão do Módulo do ASP.NET Core instalado:

1. No sistema de hospedagem, navegue até *%windir%\System32\inetsrv*.
1. Localize o arquivo *aspnetcore.dll*.
1. Clique com o botão direito do mouse no arquivo e selecione **Propriedades** no menu contextual.
1. Selecione a guia **Detalhes**. A **Versão do arquivo** e a **Versão do produto** representam a versão instalada do módulo.

Os logs de instalador do pacote de hospedagem para o módulo são encontrados em *C:\\Usuários\\%UserName%\\AppData\\Local\\Temp*. O arquivo é nomeado *dd_DotNetCoreWinSvrHosting__\<carimbo de data/hora>_000_AspNetCoreModule_x64.log*.

## <a name="module-schema-and-configuration-file-locations"></a>Locais dos arquivos de módulo, de esquema e de configuração

### <a name="module"></a>Módulo

**IIS (x86/amd64):**

   * %windir%\System32\inetsrv\aspnetcore.dll

   * %windir%\SysWOW64\inetsrv\aspnetcore.dll

**IIS Express (x86/amd64):**

   * %ProgramFiles%\IIS Express\aspnetcore.dll

   * %ProgramFiles(x86)%\IIS Express\aspnetcore.dll

### <a name="schema"></a>Esquema

**IIS**

   * %windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml

**IIS Express**

   * %ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml

### <a name="configuration"></a>Configuração

**IIS**

   * %windir%\System32\inetsrv\config\applicationHost.config

**IIS Express**

   * .vs\config\applicationHost.config

Os arquivos podem ser encontrados pesquisando por *aspnetcore.dll* no arquivo *applicationHost.config*. Para o IIS Express, o arquivo *applicationHost.config* não existirá por padrão. O arquivo é criado em *\<application_root>\\.vs\\config* ao iniciar qualquer projeto de aplicativo Web na solução do Visual Studio.
