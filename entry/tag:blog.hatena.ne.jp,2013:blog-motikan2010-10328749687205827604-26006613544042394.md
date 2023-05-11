<div style="text-align: center;">[f:id:motikan2010:20200410223604p:plain:w400]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　<span><a href="https://docs.microsoft.com/ja-jp/cli/azure/get-started-with-azure-cli?view=azure-cli-latest" target="_blank">Azure CLIコマンド(az)</a></span>でAzure Application Gatewayの起動と停止をする機会があったのでメモ。  
ちなみにWebコンソールからは現時点ではできないみたいです。  

## 利用するバージョン

　利用している Azure CLI のバージョンは以下の通りです。  
<div class="sm-code">
```
$ az version
This command is in preview. It may be changed/removed in a future release.
{
  "azure-cli": "2.1.0",
  "azure-cli-command-modules-nspkg": "2.0.3",
  "azure-cli-core": "2.1.0",
  "azure-cli-nspkg": "3.0.4",
  "azure-cli-telemetry": "1.0.4",
  "extensions": {}
}
```
</div>

　Application Gateway制御のヘルプは以下の通りです。  
本記事では`stop`と`start`について説明します。
<div class="sm-code">
```
az network application-gateway -h

Group
    az network application-gateway : Manage application-level routing and load balancing services.
        To learn more about Application Gateway, visit https://docs.microsoft.com/azure/application-
        gateway/application-gateway-create-gateway-cli.

Subgroups:
    address-pool            : Manage address pools of an application gateway.
    auth-cert               : Manage authorization certificates of an application gateway.
    frontend-ip             : Manage frontend IP addresses of an application gateway.
    frontend-port           : Manage frontend ports of an application gateway.
    http-listener           : Manage HTTP listeners of an application gateway.
    http-settings           : Manage HTTP settings of an application gateway.
    identity                : Manage the managed service identity of an application gateway.
    probe                   : Manage probes to gather and evaluate information on a gateway.
    redirect-config         : Manage redirect configurations.
    rewrite-rule            : Manage rewrite rules of an application gateway.
    root-cert               : Manage trusted root certificates of an application gateway.
    rule                    : Evaluate probe information and define routing rules.
    ssl-cert                : Manage SSL certificates of an application gateway.
    ssl-policy              : Manage the SSL policy of an application gateway.
    url-path-map            : Manage URL path maps of an application gateway.
    waf-config [Deprecated] : Configure the settings of a web application firewall.
    waf-policy              : Manage application gateway web application firewall (WAF) policies.

Commands:
    create                  : Create an application gateway.
    delete                  : Delete an application gateway.
    list                    : List application gateways.
    show                    : Get the details of an application gateway.
    show-backend-health     : Get information on the backend health of an application gateway.
    start                   : Start an application gateway. 👈 本記事で紹介するサブコマンド
    stop                    : Stop an application gateway.  👈 本記事で紹介するサブコマンド
    update                  : Update an application gateway.
    wait                    : Place the CLI in a waiting state until a condition of the application
                              gateway is met.
```
</div>


## 検証

　停止・起動には「サブスクリプションID」、「リソースグループ名」、「アプリケーションゲートウェイ名」を指定する必要があります。  
停止・起動する対象のリソースの情報は以下の通りです。  

- サブスクリプションID：`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- リソースグループ名：`testResourceGroup`
- アプリケーションゲートウェイ名：`testAppGateway`

### Application Gateway の状態を確認（az network application-gateway list）

　`list`を指定することで状態を確認することができます。  

作成時にこのコマンドを実行したので、状態は「`Running`」で起動していることが分かります。  

停止方法については後述しますが、停止状態の場合は「`Stopped`」となります。
```
$ az network application-gateway list | \
  jq -r '.[] | [.name, .operationalState, .id] | @csv'
"testAppGateway","Running","/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.Network/applicationGateways/testAppGateway"
```

### Application Gateway の停止（az network application-gateway stop）

　以下は`stop`を指定時のヘルプです。  
<div class="sm-code">
```
$ az network application-gateway stop -h

Command
    az network application-gateway stop : Stop an application gateway.

Arguments

Resource Id Arguments
    --ids               : One or more resource IDs (space-delimited). It should be a complete
                          resource ID containing all information of 'Resource Id' arguments. If
                          provided, no other 'Resource Id' arguments should be specified.
    --name -n           : Name of the application gateway.
    --resource-group -g : Name of resource group. You can configure the default group using `az
                          configure --defaults group=<name>`.
    --subscription      : Name or ID of subscription. You can configure the default subscription
                          using `az account set -s NAME_OR_ID`.

Global Arguments
    --debug             : Increase logging verbosity to show all debug logs.
    --help -h           : Show this help message and exit.
    --output -o         : Output format.  Allowed values: json, jsonc, none, table, tsv, yaml,
                          yamlc.  Default: json.
    --query             : JMESPath query string. See http://jmespath.org/ for more information and
                          examples.
    --verbose           : Increase logging verbosity. Use --debug for full debug logs.

Examples
    Stop an application gateway.
        az network application-gateway stop -g MyResourceGroup -n MyAppGateway
```
</div>

　停止は以下のコマンドを実行します。
```
$ az network application-gateway stop \
    --id '/subscriptions/<Subscription ID>/resourceGroups/<Resource Groups>/providers/Microsoft.Network/applicationGateways/<Application Gateway Name>'
```

　実際に停止コマンドを実行し、状態を確認してみます。
```
-- 停止コマンドを実行
$ az network application-gateway stop \
    --id '/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.Network/applicationGateways/testAppGateway'

-- 状態確認
$ az network application-gateway list | \
  jq -r '.[] | [.name, .operationalState, .id] | @csv'
"testAppGateway","Stopped","/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.Network/applicationGateways/testAppGateway"
```
　無事停止されていることが確認できました。  

### Application Gateway の起動（az network application-gateway start）

　以下は`start`を指定時のヘルプです。
<div class="sm-code">
```
$ az network application-gateway start -h

Command
    az network application-gateway start : Start an application gateway.

Arguments

Resource Id Arguments
    --ids               : One or more resource IDs (space-delimited). It should be a complete
                          resource ID containing all information of 'Resource Id' arguments. If
                          provided, no other 'Resource Id' arguments should be specified.
    --name -n           : Name of the application gateway.
    --resource-group -g : Name of resource group. You can configure the default group using `az
                          configure --defaults group=<name>`.
    --subscription      : Name or ID of subscription. You can configure the default subscription
                          using `az account set -s NAME_OR_ID`.

Global Arguments
    --debug             : Increase logging verbosity to show all debug logs.
    --help -h           : Show this help message and exit.
    --output -o         : Output format.  Allowed values: json, jsonc, none, table, tsv, yaml,
                          yamlc.  Default: json.
    --query             : JMESPath query string. See http://jmespath.org/ for more information and
                          examples.
    --verbose           : Increase logging verbosity. Use --debug for full debug logs.

Examples
    Start an application gateway.
        az network application-gateway start -g MyResourceGroup -n MyAppGateway

For more specific examples, use: az find "az network application-gateway start"
```
</div>

　実際に起動コマンドを実行します。数分後起動していることが確認できます。
```
$ az network application-gateway start \
    --id '/subscriptions/<Subscription ID>/resourceGroups/<Resource Groups>/providers/Microsoft.Network/applicationGateways/<Policy Name>'
```



