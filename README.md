# アーキテクチャー
今回のアーキテクチャーは以下の通りです。

00.png

簡単な流れは以下の通りです。

1. クライアント PC 上で危険なアクティビティを Microsoft Defender for Endpoint が検出する
1. Microsoft Defender for Endpoint のインシデントが Microsoft Sentinel に同期され、インシデントが自動作成される
1. Microsoft Sentinel に自動構成されたオートメーションルールにより、Logic Apps フローが実行される
1. Logic Apps フロー内で、Azure OpenAI Service が HTTP コールされ、Sentinel インシデントの件名と説明が翻訳、内容の要約を返す
1. Logic Apps フローで生成されたデータが Microsoft Teams アダプティブカードとして送信される

# 構成手順
以下を構成、実装する事で動作確認用環境の構成が可能です。

## 事前準備
### Microsoft Defender for Endpoint 評価ラボの構成
1. [手順ページ](https://learn.microsoft.com/ja-jp/microsoft-365/security/defender-endpoint/evaluation-lab?view=o365-worldwide)を参考に評価ラボ（仮想マシン）を構成します。

1. [サンプルインシデントに関するドキュメント](https://wcddocsprdeus.file.core.windows.net/sevillestaticfiles/AttackSimulationDIYv4_FileAttack.pdf?sv=2022-11-02&se=2023-09-25T13%3A24%3A02Z&sr=f&sp=r&sig=JctGVykFl4EGxSOklvDwL%2FIYc1lSWXlR7dw9G5HzcI0%3D)を確認し、ダウンロードしておきます。

[!Warning]

組織の EDR ソリューションに検閲される可能性がありますので、絶対にローカル PC 上ではファイルを開かないで下さい。

### Microsoft Sentinel の構成
1. [Microsoft Sentinel を有効](https://learn.microsoft.com/ja-jp/azure/sentinel/quickstart-onboard#enable-microsoft-sentinel-)にする。

1. [手順ページ](https://learn.microsoft.com/ja-jp/azure/sentinel/quickstart-onboard#install-a-solution-from-the-content-hub)に則り、"Microsoft Defender for Cloud"と"Microsoft Defender for Endpoint"ソリューションをインストールする。

1. [手順ページ](https://learn.microsoft.com/ja-jp/azure/sentinel/create-incidents-from-alerts#using-microsoft-security-incident-creation-analytics-rules)に則り、"Microsoft Defender for Cloud"と"Microsoft Defender for Endpoint"を"Microsoft security service"に設定し、ルールを作成する。

### リソースの作成
