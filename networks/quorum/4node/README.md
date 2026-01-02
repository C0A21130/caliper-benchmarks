# GoQuorumネットワーク
[Caliper](https://github.com/hyperledger/caliper/) の必要な前提条件をすべて満たしていることを確認してください。

## ベンチマークの環境構築
GoQuorumネットワークで caliper-benchmark を使用する環境をセットアップする手順を説明する。
- この環境用のディレクトリを作成します (例: `mkdir besu-benchmarks`)
- そのディレクトリに移動します (例: `cd besu-benchmarks`)
- caliper-benchmarks をクローンします (上で作成したディレクトリ内に作成されていることを確認してください)
```bash
git clone https://github.com/C0A21130/caliper-benchmarks.git
```
- caliper-benchmarks ディレクトリに移動します
```bash
cd caliper-benchmarks
```
- 最新版の Caliper をインストールします
```bash
npm  install  --only=prod  @hyperledger/caliper-cli
```
- Caliper を Besu の最新バージョンにバインドします
```bash
npx caliper bind --caliper-bind-sut besu:latest
```

## ベンチマークの実行
必ず `caliper-benchmarks` ディレクトリ内で作業してください。実行したいシナリオに応じて、以下のコマンド例を使用します。

**注意点1**
まず `hardhat` 等を利用してスマートコントラクトをデプロイしてください。
デプロイ後に[networkconfig.json](/networks/quorum/4node/networkconfig.json)ファイルにおけるスマートコントラクトアドレスである `ethereum.contracts.ERC-721.address` と `ethereum.contracts.SsdlabToken.address` を変更してください。
事前にスマートコントラクトをデプロイするためインストールはスキップします。

**注意点2**
毎回のベンチマーク実行時にスマートコントラクトを再デプロイする必要があります。
状態を管理する `simple-state.js` により、NFT転送のベンチマークの実行時にトークンIDが0に初期化されてしまいます。
そのためベンチマーク実行時は、毎回スマートコントラクトを再デプロイする必要があります。

**ベンチマークの実行**

- ERC-721スマートコントラクト
```bash
npx caliper launch manager --caliper-benchconfig benchmarks/scenario/ERC-721/config.yaml --caliper-networkconfig networks/quorum/4node/networkconfig.json --caliper-workspace . --caliper-flow-skip-install
```

- Scoringによる認可機能をERC-721スマートコントラクト
```bash
npx caliper launch manager --caliper-benchconfig benchmarks/scenario/scoring/config.yaml --caliper-networkconfig networks/quorum/4node/networkconfig.json --caliper-workspace . --caliper-flow-skip-install
```

### Config network
ネットワーク設定は [networkconfig.json](networkconfig.json) で定義されています。

- **ブロックチェーンの種類**: Ethereum
- **Network URL**: `ws://10.203.92.69:32000`
- **Contract Deployer Address**: `0xd1cf9d73a91de6630c2bb068ba5fddf9f0deac09`
- **Transaction Confirmation Blocks**: 2
- **スマートコントラクト設定**:

| 項目 | ERC-721 | SsdlabToken (Scoring) |
|------|---------|----------------------|
| **Path** | `./src/ethereum/ERC-721/ERC-721.json` | `./src/ethereum/scoring/SsdlabToken.json` |
| **Address** | `0x9EEa1CE7c0110036A1c89d3927493818F601307D` | `0x64ddbde36fD0d3E5fA4F0dF07aB02DA597f2987F` |
| **Estimate Gas** | `true` | `true` |
| **Gas (mint/safeMint)** | 100000 | 150000 |
| **Gas (transferFrom)** | 200000 | 700000 |

### Config benchmark
ベンチマーク設定は [config.yaml](../../../benchmarks/scenario/scoring/config.yaml) で定義されています。

- **Name**: Scoring benchmark
- **Description**: トラストスコアリングを用いた認可機能を持つNFT取引スマートコントラクトのベンチマーク
- **Workers**: 1
- **ベンチマーク**:

| 項目 | Mint(Scoring) | Transfer(Scoring) |
|------|---------------|-------------------|
| **説明** | NFTをミント | NFTをアカウント間で転送 |
| **トランザクション数** | 2000 | 2000 |
| **レート制御** | fixed-rate | fixed-rate |
| **TPS** | 20 | 20 |
| **ワークロードモジュール** | `benchmarks/scenario/scoring/mint.js` | `benchmarks/scenario/scoring/transfer.js` |
| **呼び出しメソッド** | `safeMint` | `transferFrom` |
