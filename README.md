# GitHub Secret scanning's push protection demo

GitHub Secret scanningのpush protectionのデモリポジトリです。

## GitHub Secret scanningの解説

GitHub Secret scanningは、GitHub上にpushされたコードの中に、秘密情報が含まれていないかを検出する機能です。

検知される秘密情報のパターンは、パートナープログラムによって登録されたサービスプロバイダが提供するパターンのほか、ユーザーが独自に定義する（※）こともできます。（※カスタムパターンについては、[シークレット スキャンのカスタム パターンの定義](https://docs.github.com/ja/enterprise-cloud@latest/code-security/secret-scanning/defining-custom-patterns-for-secret-scanning)をご参照ください）

パートナープログラムでは、サービスプロバイダ（GitHub、Microsoft、Amazon Web Servicesなどのシークレットの発行元）がGitHubに対しシークレットパターンを申請することで、GitHub Secret scanningでそのシークレットパターンが検知・対処されるようになります。パートナープログラムによりサポートされるシークレットパターンについては、[サポートされているシークレット](https://docs.github.com/ja/code-security/secret-scanning/secret-scanning-patterns#supported-secrets)をご参照ください。

```mermaid
sequenceDiagram
  participant G as GitHub
  participant P as サービスプロバイダ
  P->>G: シークレットパターンを申請する
  G-->>P: シークレットパターンを受理する
```

### シークレットが検知された場合の挙動

GitHub Secret scanningによりシークレットパターンが検知されると、GitHubはサービスプロバイダに対し、シークレットが漏洩されたことを通知します。サービスプロバイダは、シークレットを無効化するとともに、プロバイダのユーザーに対し、シークレットの漏洩を通知します。

```mermaid
sequenceDiagram
  participant GU as GitHubリポジトリのユーザー
  participant G as GitHub
  participant P as サービスプロバイダ
  participant PU as プロバイダのユーザー
  GU->>G: コードをpushする
  G->>G: リポジトリをスキャンする
  alt シークレットパターンを検知した場合
    Note left of G: シークレットを検知する
    G->>P: シークレットが漏洩されたことを通知する
    P->>P: シークレットを無効化する
    P->>PU: シークレットの漏洩を通知する
    alt アラートの通知を有効にしている場合
      G->>GU: シークレットが検知されたことを通知する
    end
  end
```

#### アラートの通知を有効にしている場合

何も設定していない場合、シークレットの検知はGitHubのリポジトリの管理者に通知されませんが、リポジトリの設定で「Receive alerts on GitHub for detected secrets, keys, or other tokens.」を有効にしていれば、そのリポジトリの管理者へも通知されます。

![リポジトリ設定のSecret scanningの設定](./docs/images/secret-scanning_to-enable.png)

### push protection（プッシュ保護）

アラートの通知を有効にした上で「Push protection」も有効にすると、`git push`を実行したとき、その内容がスキャンされ、シークレットパターンが検出された場合にpushを拒否できるようになります。

![Secret scanningのPush protectionを有効にする](./docs/images/secret-scanning_to-enable-push-protection.png)

```mermaid
sequenceDiagram
  participant GU as GitHubリポジトリのユーザー
  participant G as GitHub
  participant P as サービスプロバイダ
  participant PU as プロバイダのユーザー
  alt push protectionを有効にしている場合
    GU->>G: コードをpushしようとする
    G->>G: pushされる内容をスキャンする
    alt シークレットパターンを検知した場合
      Note left of G: シークレットを検知する
      G-->>GU: シークレットパターンが含まれているため、エラーを返す
    end
  end
```
