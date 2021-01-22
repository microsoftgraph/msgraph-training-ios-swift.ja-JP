<!-- markdownlint-disable MD002 MD041 -->

この演習では、前の演習のアプリケーションを拡張して、Azure AD での認証をサポートします。 これは、Microsoft Graph を呼び出すのに必要な OAuth アクセス トークンを取得するために必要です。 これを行うには [、iOS 用 Microsoft Authentication Library (MSAL) を](https://github.com/AzureAD/microsoft-authentication-library-for-objc) アプリケーションに統合します。

1. **AuthSettings.plist** という名前の **GraphTu graphl** プロジェクトに新しいプロパティ リスト ファイルを作成します。 
1. ルート ディクショナリ内のファイルに次の項目 **を** 追加します。

    | キー | 型 | 値 |
    |-----|------|-------|
    | `AppId` | String | Azure Portal からのアプリケーション ID |
    | `GraphScopes` | 配列 | 3 つの文字列値: `User.Read` `MailboxSettings.Read` 、 、 `Calendars.ReadWrite` |

    ![Xcode の AuthSettings.plist ファイルのスクリーンショット](images/auth-settings.png)

> [!IMPORTANT]
> git などのソース管理を使用している場合は **、AuthSettings.plist** ファイルをソース管理から除外して、アプリ ID が誤って漏洩しないようにする良い時期です。

## <a name="implement-sign-in"></a>サインインの実装

このセクションでは、MSAL のプロジェクトを構成し、認証マネージャー クラスを作成し、サインインしてサインアウトするアプリを更新します。

### <a name="configure-project-for-msal"></a>MSAL 用にプロジェクトを構成する

1. プロジェクトの機能に新しいキーチェーン グループを追加します。
    1. **GraphTu読み込みプロジェクトを** 選択し、次に **[&機能] を選択します**。
    1. [+ **機能] を** 選択し、[キーチェーンの共有] **をダブルクリックします**。
    1. 値を持つキーチェーン グループを追加します `com.microsoft.adalcache` 。

1. **[Info.plist] をクリックし、[** ファイル名を指定して **開く]、次** に [ソース コード]**の順に選択します**。
1. 要素内に次を追加 `<dict>` します。

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
      </dict>
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. **AppDelegate.swift を開** き、ファイルの上部に次の import ステートメントを追加します。

    ```Swift
    import MSAL
    ```

1. 次の関数を `AppDelegate` クラスに追加します。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>認証マネージャーを作成する

1. AuthenticationManager.swift **という名前の** **GraphTu graphl** プロジェクトに新しい **Swift ファイルを作成します**。 次のコードをファイルに追加します。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>サインインとサインアウトを追加する

1. **SignInViewController.swift を開き**、その内容を次のコードに置き換えます。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. **WelcomeViewController.swift を開き**、既存の `signOut` 関数を次の関数に置き換える。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. 変更を保存し、シミュレーターでアプリケーションを再起動します。

アプリにサインインすると、Xcode の出力ウィンドウにアクセス トークンが表示されます。

![アクセス トークンを示す Xcode の出力ウィンドウのスクリーンショット](images/access-token-output.png)

## <a name="get-user-details"></a>ユーザーの詳細情報を取得する

このセクションでは、Microsoft Graph へのすべての呼び出しを保持するヘルパー クラスを作成し、この新しいクラスを使用してログイン ユーザーを取得するために更新 `WelcomeViewController` します。

1. GraphManager.swift **という名前の** **GraphTu graphl** プロジェクトに新しい **Swift ファイルを作成します**。 次のコードをファイルに追加します。

    ```Swift
    import Foundation
    import MSGraphClientSDK
    import MSGraphClientModels

    class GraphManager {

        // Implement singleton pattern
        static let instance = GraphManager()

        private let client: MSHTTPClient?

        public var userTimeZone: String

        private init() {
            client = MSClientFactory.createHTTPClient(with: AuthenticationManager.instance)
            userTimeZone = "UTC"
        }

        public func getMe(completion: @escaping(MSGraphUser?, Error?) -> Void) {
            // GET /me
            let select = "$select=displayName,mail,mailboxSettings,userPrincipalName"
            let meRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me?\(select)")!)
            let meDataTask = MSURLSessionDataTask(request: meRequest, client: self.client, completion: {
                (data: Data?, response: URLResponse?, graphError: Error?) in
                guard let meData = data, graphError == nil else {
                    completion(nil, graphError)
                    return
                }

                do {
                    // Deserialize response as a user
                    let user = try MSGraphUser(data: meData)
                    completion(user, nil)
                } catch {
                    completion(nil, error)
                }
            })

            // Execute the request
            meDataTask?.execute()
        }
    }
    ```

1. **WelcomeViewController.swift を** 開き、ファイルの上部に `import` 次のステートメントを追加します。

    ```Swift
    import MSGraphClientModels
    ```

1. 次のプロパティを `WelcomeViewController` クラスに追加します。

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. 既存のコードを次 `viewDidLoad` のコードに置き換えます。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

変更を保存してアプリを今すぐ再起動すると、サインイン後にユーザーの表示名とメール アドレスで UI が更新されます。
