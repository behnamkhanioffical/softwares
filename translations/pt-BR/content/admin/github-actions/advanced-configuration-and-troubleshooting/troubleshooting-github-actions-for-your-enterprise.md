---
title: Solucionar problemas no GitHub Actions para a sua empresa
intro: 'Solucionar problemas comuns que ocorrem ao usar {% data variables.product.prodname_actions %} em {% data variables.product.prodname_ghe_server %}.'
permissions: 'Site administrators can troubleshoot {% data variables.product.prodname_actions %} issues and modify {% data variables.product.prodname_ghe_server %} configurations.'
versions:
  ghes: '*'
type: how_to
topics:
  - Actions
  - Enterprise
  - Troubleshooting
redirect_from:
  - /admin/github-actions/troubleshooting-github-actions-for-your-enterprise
shortTitle: Solução de problemas do GitHub Actions
---

## Verificando a saúde de {% data variables.product.prodname_actions %}

Você pode verificar a saúde de {% data variables.product.prodname_actions %} em {% data variables.product.product_location %} com o utilitário da linha de comando `ghe-actions-check`. Para obter mais informações, consulte "[Utilitários de linha de comando](/admin/configuration/configuring-your-enterprise/command-line-utilities#ghe-actions-check)" e "[Acessando o shell administrativo (SSH)](/admin/configuration/configuring-your-enterprise/accessing-the-administrative-shell-ssh)".

## Configurar executores auto-hospedados ao usar um certificado autoassinado por {% data variables.product.prodname_ghe_server %}

{% data reusables.actions.enterprise-self-signed-cert %} Para obter mais informações, consulte "[Configurar TLS](/admin/configuration/configuring-tls)".

### Instalar o certificado na máquina do executor

Para um executor auto-hospedado conectar-se a um {% data variables.product.prodname_ghe_server %} usando um certificado autoassinado, você deverá instalar o certificado na máquina do executor para que a conexão seja mais rígida.

Para as etapas necessárias para instalar um certificado, consulte a documentação do sistema operacional do seu executor.

### Configurar o Node.JS para usar o certificado

A maioria das ações são escritas em JavaScript e são executadas usando Node.js, que não usa o armazenamento de certificados do sistema operacional. Para o aplicativo runner auto-hospedado usar o certificado, você deve definir a variável de ambiente `NODE_EXTRA_CA_CERTS` na máquina do executor.

Você pode definir a variável de ambiente como uma variável de ambiente do sistema, ou declará-la em um arquivo denominado _.env_ no diretório do aplicativo do executor auto-hospedado.

Por exemplo:

```shell
NODE_EXTRA_CA_CERTS=/usr/share/ca-certificates/extra/mycertfile.crt
```

As variáveis de ambiente são lidas quando o aplicativo do executor auto-hospedado é iniciado. Portanto, você deve definir a variável de ambiente antes de configurar ou iniciar o aplicativo do executor auto-hospedado. Se a sua configuração de certificado for alterada, você deverá reiniciar o aplicativo do executor auto-hospedado.

### Configurar contêineres do Docker para usar o certificado

Se você usa ações do contêiner do Docker ou contêineres de serviço nos seus fluxos de trabalho, você também deverá instalar o certificado na sua imagem do Docker, além de definir a variável de ambiente acima.

## Configurar as definições de proxy HTTP para {% data variables.product.prodname_actions %}

{% data reusables.actions.enterprise-http-proxy %}

Se estas configurações não estiverem definidas corretamente, você poderá receber erros como `Recurso movido inesperadamente para https://<IP_ADDRESS>` ao definir ou mudar a configuração de {% data variables.product.prodname_actions %}.

## Runners not connecting to {% data variables.product.prodname_ghe_server %} with a new hostname

{% data reusables.enterprise_installation.changing-hostname-not-supported %}

If you deploy {% data variables.product.prodname_ghe_server %} in your environment with a new hostname and the old hostname no longer resolves to your instance, self-hosted runners will be unable to connect to the old hostname, and will not execute any jobs.

You will need to update the configuration of your self-hosted runners to use the new hostname for {% data variables.product.product_location %}. Each self-hosted runner will require one of the following procedures:

* No diretório de do aplicativo do executor auto-hospedado, edite os arquivos de `.runner` e `.credentials` para substituir todas as menções do nome de host antigo pelo novo nome de host. Em seguida, reinicie o aplicativo do executor auto-hospedado.
* Remova o executor de {% data variables.product.prodname_ghe_server %} usando a interface do usuário e adicione-o novamente. Para obter mais informações, consulte "[Removendo os executores auto-hospedados](/actions/hosting-your-own-runners/removing-self-hosted-runners)" e "[Adicionando executores auto-hospedados](/actions/hosting-your-own-runners/adding-self-hosted-runners)".

## Trabalhos travados e limites de CPU e de memória das {% data variables.product.prodname_actions %}

{% data variables.product.prodname_actions %} is composed of multiple services running on {% data variables.product.product_location %}. By default, these services are set up with default CPU and memory limits that should work for most instances. However, heavy users of {% data variables.product.prodname_actions %} might need to adjust these settings.

You may be hitting the CPU or memory limits if you notice that jobs are not starting (even though there are idle runners), or if the job's progress is not updating or changing in the UI.

### 1. Verifique o uso total da CPU e memória no console de gerenciamento

Access the management console and use the monitor dashboard to inspect the overall CPU and memory graphs under "System Health". For more information, see "[Accessing the monitor dashboard](/admin/enterprise-management/accessing-the-monitor-dashboard)."

If the overall "System Health" CPU usage is close to 100%, or there is no free memory left, then {% data variables.product.product_location %} is running at capacity and needs to be scaled up. For more information, see "[Increasing CPU or memory resources](/admin/enterprise-management/increasing-cpu-or-memory-resources)."

### 2. Verifique o uso de CPU e a memória dos trabalhos Nomad no console de gerenciamento

If the overall "System Health" CPU and memory usage is OK, scroll down the monitor dashboard page to the "Nomad Jobs" section, and look at the "CPU Percent Value" and "Memory Usage" graphs.

Each plot in these graphs corresponds to one service. For {% data variables.product.prodname_actions %} services, look for:

* `mps_frontend`
* `mps_backend`
* `token_frontend`
* `token_backend`
* `actions_frontend`
* `actions_backend`

If any of these services are at or near 100% CPU utilization, or the memory is near their limit (2 GB by default), then the resource allocation for these services might need increasing. Take note of which of the above services are at or near their limit.

### 3. Aumenta a alocação de recursos para serviços em seu limite

1. Efetue o login no shell administrativo usando SSH. Para obter mais informações, consulte "[Acessar o shell administrativo (SSH)](/admin/configuration/accessing-the-administrative-shell-ssh)".
1. Execute o comando a seguir para ver quais recursos estão disponíveis para alocação:

   ```shell
   nomad node status -self
   ```

   Na saída, encontre a seção "Recursos alocados". É algo parecido com o exemplo a seguir:

   ```
   Allocated Resources
   CPU              Memory          Disk
   7740/49600 MHZ   23 GiB/32 GiB   4.4 GiB/7.9 GiB
   ```

   Para a memória e a CPU, isso mostra quanto é alocado para o **total** de **todos** serviços (o valor à esquerda) e quanto está disponível (o valor correto). No exemplo acima, há 23 GiB de memória alocado para um total de 32 GiB. Isto significa que há 9 GiB de memória disponíveis para atribuição.

   {% warning %}

   **Aviso:** Tenha cuidado para não alocar mais do que o total de recursos disponíveis, ou os serviços não poderão ser iniciados.

   {% endwarning %}
1. Mude o diretório para `/etc/consul-templates/etc/nomad-jobs/ações`:

   ```shell
   cd /etc/consul-templates/etc/nomad-jobs/actions
   ```

   Neste diretório existem três arquivos que correspondem aos serviços de {% data variables.product.prodname_actions %} descritos anteriormente:

   * `mps.hcl.ctmpl`
   * `token.hcl.ctmpl`
   * `actions.hcl.ctmpl`
1. Para os serviços que você identificou que precisam de ajuste, abra o arquivo correspondente e localize o grupo de `recursos` que se parece com o exemplo a seguir:

   ```
   resources {
     cpu = 512
     memory = 2048
     network {
       port "http" { }
     }
   }
   ```

   Os valores estão em MHz para recursos de CPU e em MB para recursos de memória.

   Por exemplo, para aumentar os limites de recursos no exemplo acima para 1 GHz para a CPU e 4 GB de memória, altere-os para:

   ```
   resources {
     cpu = 1024
     memory = 4096
     network {
       port "http" { }
     }
   }
   ```
1. Salve e saia do arquivo.
1. Execute o `ghe-config-apply` para aplicar as alterações.

    Ao executar `ghe-config-apply`, se você vir a saída como `Failed to run nomad job '/etc/nomad-jobs/<name>.hcl'`, a mudança provavelmente atribuiu muitos recursos de CPU ou memória. Se isso acontecer, edite os arquivos de configuração novamente e baixe a CPU ou memória alocados e execute `ghe-config-apply` novamente.
1. Depois que a configuração for aplicada, execute `ghe-actions-check` para verificar se os serviços {% data variables.product.prodname_actions %} estão operando.

{% ifversion fpt or ghec or ghes > 3.2 %}
## Solucionar problemas de falhas quando {% data variables.product.prodname_dependabot %} dispara fluxos de trabalho existentes

{% data reusables.dependabot.beta-security-and-version-updates %}

After you set up {% data variables.product.prodname_dependabot %} updates for {% data variables.product.product_location %}, you may see failures when existing workflows are triggered by {% data variables.product.prodname_dependabot %} events.

By default, {% data variables.product.prodname_actions %} workflow runs that are triggered by {% data variables.product.prodname_dependabot %} from `push`, `pull_request`, `pull_request_review`, or `pull_request_review_comment` events are treated as if they were opened from a repository fork. Unlike workflows triggered by other actors, this means they receive a read-only `GITHUB_TOKEN` and do not have access to any secrets that are normally available. This will cause any workflows that attempt to write to the repository to fail when they are triggered by {% data variables.product.prodname_dependabot %}.

There are three ways to resolve this problem:

1. Você pode atualizar seus fluxos de trabalho para que não sejam mais acionados por {% data variables.product.prodname_dependabot %} usando uma expressão como: `if: github.actor != 'dependabot[bot]'`. Para obter mais informações, consulte "[Expressões](/actions/learn-github-actions/expressions)".
2. Você pode modificar seus fluxos de trabalho para usar um processo de duas etapas que inclui `pull_request_target` que não tem essas limitações. Para obter mais informações, consulte "[Automatizando {% data variables.product.prodname_dependabot %} com {% data variables.product.prodname_actions %}](/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically/automating-dependabot-with-github-actions#responding-to-events)".
3. Você pode fornecer fluxos de trabalho acionados pelo acesso de {% data variables.product.prodname_dependabot %} a segredos e permitir que o termo `permissões` aumente o escopo padrão do `GITHUB_TOKEN`. Para obter mais informações, consulte [Fornecendo fluxos de trabalho acionados pelo acesso{% data variables.product.prodname_dependabot %} a segredos e aumento das permissões](#providing-workflows-triggered-by-dependabot-access-to-secrets-and-increased-permissions)" abaixo.

### Fornecendo fluxos de trabalho acionados pelo acesso de {% data variables.product.prodname_dependabot %} a segredos e permissões ampliadas

1. Efetue o login no shell administrativo usando SSH. Para obter mais informações, consulte "[Acessar o shell administrativo (SSH)](/admin/configuration/accessing-the-administrative-shell-ssh)".
1. Para remover as limitações dos fluxos de trabalho acionados por {% data variables.product.prodname_dependabot %} em {% data variables.product.product_location %}, use o seguinte comando.
    ``` shell
    $ ghe-config app.actions.disable-dependabot-enforcement true
    ```
1. Aplique a configuração.
    ```shell
    $ ghe-config-apply
    ```
1. Volte para o {% data variables.product.prodname_ghe_server %}.

{% endif %}