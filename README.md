# アーキテクチャー
今回のアーキテクチャーは以下の通りです。

![00](https://github.com/TK3214-MS/POC-AppConf/assets/89323076/90290981-6f8b-4649-8f98-89cb9424ee74)

簡単な流れは以下の通りです。

1. クライアント PC 上で危険なアクティビティを Microsoft Defender for Endpoint が検出する
1. Microsoft Defender for Endpoint のインシデントが Microsoft Sentinel に同期され、インシデントが自動作成される
1. Microsoft Sentinel に自動構成されたオートメーションルールにより、Logic Apps フローが実行される
1. Logic Apps フロー内で、Azure OpenAI Service が HTTP コールされ、Sentinel インシデントの件名と説明が翻訳、内容の要約を返す
1. Logic Apps フローで生成されたデータが Microsoft Teams アダプティブカードとして送信される

# 構成手順
以下を構成、実装する事で動作確認用環境の構成が可能です。

## 事前準備
### Azure Open AI リソースの作成
1. [手順ページ](https://learn.microsoft.com/ja-jp/azure/ai-services/openai/how-to/create-resource?pivots=web-portal)に則り、Azure Open AI Service リソースを作成します。

1. [手順ページ](https://learn.microsoft.com/ja-jp/azure/ai-services/openai/how-to/create-resource?pivots=web-portal#deploy-a-model)に則り、モデルを作成します。

    ※本サンプルでは gpt-35-turbo-16k を利用しました。

### Microsoft Defender for Endpoint 評価ラボの構成
1. [手順ページ](https://learn.microsoft.com/ja-jp/microsoft-365/security/defender-endpoint/evaluation-lab?view=o365-worldwide)を参考に評価ラボ（仮想マシン）を構成します。

1. [サンプルインシデントに関するドキュメント](https://wcddocsprdeus.file.core.windows.net/sevillestaticfiles/AttackSimulationDIYv4_FileAttack.pdf?sv=2022-11-02&se=2023-09-25T13%3A24%3A02Z&sr=f&sp=r&sig=JctGVykFl4EGxSOklvDwL%2FIYc1lSWXlR7dw9G5HzcI0%3D)を確認し、ダウンロードしておきます。

    ※組織の EDR ソリューションに検閲される可能性がありますので、絶対にローカル PC 上ではファイルを開かないで下さい。

## 構成
### Microsoft Sentinel の構成
1. [Microsoft Sentinel を有効](https://learn.microsoft.com/ja-jp/azure/sentinel/quickstart-onboard#enable-microsoft-sentinel-)にする。

1. [手順ページ](https://learn.microsoft.com/ja-jp/azure/sentinel/quickstart-onboard#install-a-solution-from-the-content-hub)に則り、"Microsoft Defender for Cloud"と"Microsoft Defender for Endpoint"ソリューションをインストールする。

1. [手順ページ](https://learn.microsoft.com/ja-jp/azure/sentinel/create-incidents-from-alerts#using-microsoft-security-incident-creation-analytics-rules)に則り、"Microsoft Defender for Cloud"と"Microsoft Defender for Endpoint"を"Microsoft security service"に設定し、ルールを作成する。

### Logic Apps の構成
1. [手順ページ](https://learn.microsoft.com/ja-jp/azure/logic-apps/quickstart-create-example-consumption-workflow#create-a-consumption-logic-app-resource)に則り、従量課金モデルでの Logic App リソースを作成します。

1. [Blog 記事](https://techcommunity.microsoft.com/t5/microsoft-sentinel-blog/what-s-new-managed-identity-for-azure-sentinel-logic-apps/ba-p/2068204)を参考に、Microsoft Sentinel に関連付く Log Analytics Workspace に Logic App に有効化した Managed Identity を RBAC (Azure Sentinel Responder)設定をします。

1. 空の Logic App フローを作成し、タイマートリガーと"Microsoft Sentinel"及び"Microsoft Teams"に関連するアクションを配置し、接続情報（Sentinel には Managed Identity を選択）を設定します。

1. コードビューに切り替え"parameters"配下の接続情報をコピーします。

    ![01](https://github.com/TK3214-MS/POC-AppConf/assets/89323076/314eb350-20d9-4855-b517-15e54ecb9658)

1. 本レポジトリの"LogicApp.json"の内容をコピーし、コードビューに全て貼り付けます。

1. コピーしておいた"parameters"配下の内容と置換えます。

1. デザイナービューに戻し、以下を自身の環境値と置換えます。

    https://**`<AOAI ENDPOINT URL>`**.openai.azure.com/openai/deployments/**`<AOAI MODEL NAME>`**/chat/completions?api-version=2023-03-15-preview

1. [手順ページ](https://learn.microsoft.com/ja-jp/azure/sentinel/tutorial-respond-threats-playbook?tabs=LAC%2Cincidents)に則り、Microsoft Sentinel 自動化ルールを作成します。

    ※本サンプルでの構成例は以下の通りです。

    ![02](https://github.com/TK3214-MS/POC-AppConf/assets/89323076/c885b56a-0ee9-45ce-8320-32b697674e7c)

## 動作確認
1. 作成した評価ラボでインシデントを発生させる、もしくは[手順ページ](https://learn.microsoft.com/ja-jp/azure/defender-for-cloud/alert-validation#generate-sample-security-alerts)に則り、Microsoft Defender for Cloud サンプルアラートを生成します。

1. 以下のような通知が Microsoft Teams に送信されると構成成功です。

    ![03](https://github.com/TK3214-MS/POC-AppConf/assets/89323076/151b3664-88ea-44aa-a0f7-79b3dea61998)