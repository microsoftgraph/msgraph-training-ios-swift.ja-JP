<!-- markdownlint-disable MD002 MD041 -->

この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。 これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。 これを行うには、 [iOS 用 Microsoft Authentication Library (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-objc)をアプリケーションに統合します。

1. **Authsettings. plist**という名前の**graphtutorial**プロジェクトに、新しい**プロパティリスト**ファイルを作成します。
1. 次の項目を**ルート**ディクショナリのファイルに追加します。

    | Key | 型 | 値 |
    |-----|------|-------|
    | `AppId` | String | Azure portal からのアプリケーション ID |
    | `GraphScopes` | 配列 | 2つの文字列`User.Read`値:`Calendars.Read` |

    ![Xcode の AuthSettings plist ファイルのスクリーンショット](./images/auth-settings.png)

> [!IMPORTANT]
> Git などのソース管理を使用している場合は、アプリ ID が誤ってリークしないように、ソース管理から**Authsettings の設定**ファイルを除外することをお勧めします。

## <a name="implement-sign-in"></a>サインインの実装

このセクションでは、MSAL のプロジェクトを構成し、認証マネージャークラスを作成し、サインインしてサインアウトするためにアプリを更新します。

### <a name="configure-project-for-msal"></a>MSAL のプロジェクトを構成する

1. [コントロール] をクリックし、[**名前を付けて開く**]、[**ソースコード**] の順に選択し**ます。**
1. `<dict>`要素内に次のように追加します。

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
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. **Appdelegate**を開き、ファイルの先頭に次の import ステートメントを追加します。

    ```Swift
    import MSAL
    ```

1. 次の関数を `AppDelegate` クラスに追加します。

    ```Swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        guard let sourceApplication = options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String else {
            return false
        }

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApplication)
    }
    ```

### <a name="create-authentication-manager"></a>認証マネージャーを作成する

1. 「 **Authenticationmanager. Swift**」という名前の**graphtutorial**プロジェクトに新しい**Swift ファイル**を作成します。 このファイルに次のコードを追加します。

    ```Swift
    import Foundation
    import MSAL

    class AuthenticationManager {

        // Implement singleton pattern
        static let instance = AuthenticationManager()

        private let publicClient: MSALPublicClientApplication?
        private let appId: String
        private let graphScopes: Array<String>

        private init() {
            // Get app ID and scopes from AuthSettings.plist
            let bundle = Bundle.main
            let authConfigPath = bundle.path(forResource: "AuthSettings", ofType: "plist")!
            let authConfig = NSDictionary(contentsOfFile: authConfigPath)!

            self.appId = authConfig["AppId"] as! String
            self.graphScopes = authConfig["GraphScopes"] as! Array<String>

            do {
                // Create the MSAL client
                try self.publicClient = MSALPublicClientApplication(clientId: self.appId,
                                                                    keychainGroup: bundle.bundleIdentifier)
            } catch {
                print("Error creating MSAL public client: \(error)")
                self.publicClient = nil
            }
        }

        public func getPublicClient() -> MSALPublicClientApplication? {
            return self.publicClient
        }

        public func getTokenInteractively(completion: @escaping(_ accessToken: String?, Error?) -> Void) {
            // Call acquireToken to open a browser so the user can sign in
            publicClient?.acquireToken(forScopes: self.graphScopes, completionBlock: {
                (result: MSALResult?, error: Error?) in
                guard let tokenResult = result, error == nil else {
                    print("Error getting token interactively: \(String(describing: error))")
                    completion(nil, error)
                    return
                }

                print("Got token interactively: \(tokenResult.accessToken)")
                completion(tokenResult.accessToken, nil)
            })
        }

        public func getTokenSilently(completion: @escaping(_ accessToken: String?, Error?) -> Void) {
            // Check if there is an account in the cache
            var userAccount: MSALAccount?

            do {
                userAccount = try publicClient?.allAccounts().first
            } catch {
                print("Error getting account: \(error)")
            }

            if (userAccount != nil) {
                // Attempt to get token silently
                publicClient?.acquireTokenSilent(forScopes: self.graphScopes, account: userAccount!, completionBlock: {
                    (result: MSALResult?, error: Error?) in
                    guard let tokenResult = result, error == nil else {
                        print("Error getting token silently: \(String(describing: error))")
                        completion(nil, error)
                        return
                    }

                    print("Got token silently: \(tokenResult.accessToken)")
                    completion(tokenResult.accessToken, nil)
                })
            } else {
                print("No account in cache")
                completion(nil, NSError(domain: "AuthenticationManager",
                                        code: MSALErrorCode.interactionRequired.rawValue, userInfo: nil))
            }
        }

        public func signOut() -> Void {
            do {
                // Remove all accounts from the cache
                let accounts = try publicClient?.allAccounts()

                try accounts!.forEach({
                    (account: MSALAccount) in
                    try publicClient?.remove(account)
                })
            } catch {
                print("Sign out error: \(String(describing: error))")
            }
        }
    }
    ```

### <a name="add-sign-in-and-sign-out"></a>サインインとサインアウトを追加する

1. **SignInViewController**を開き、そのコンテンツを次のコードに置き換えます。

    ```Swift
    import UIKit

    class SignInViewController: UIViewController {

        private let spinner = SpinnerViewController()

        override func viewDidLoad() {
            super.viewDidLoad()
            // Do any additional setup after loading the view.

            // See if a user is already signed in
            spinner.start(container: self)

            AuthenticationManager.instance.getTokenSilently {
                (token: String?, error: Error?) in

                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let _ = token, error == nil else {
                        // If there is no token or if there's an error,
                        // no user is signed in, so stay here
                        return
                    }

                    // Since we got a token, a user is signed in
                    // Go to welcome page
                    self.performSegue(withIdentifier: "userSignedIn", sender: nil)
                }
            }
        }

        @IBAction func signIn() {
            spinner.start(container: self)

            // Do an interactive sign in
            AuthenticationManager.instance.getTokenInteractively {
                (token: String?, error: Error?) in

                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let _ = token, error == nil else {
                        // Show the error and stay on the sign-in page
                        let alert = UIAlertController(title: "Error signing in",
                                                      message: error.debugDescription,
                                                      preferredStyle: .alert)

                        alert.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
                        self.present(alert, animated: true)
                        return
                    }

                    // Signed in successfully
                    // Go to welcome page
                    self.performSegue(withIdentifier: "userSignedIn", sender: nil)
                }
            }
        }
    }
    ```

1. **WelcomeViewController**を開き、既存`signOut`の関数を次のように置き換えます。

    ```Swift
    @IBAction func signOut() {
        AuthenticationManager.instance.signOut()
        self.performSegue(withIdentifier: "userSignedOut", sender: nil)
    }
    ```

1. 変更を保存し、シミュレータでアプリケーションを再起動します。

アプリにサインインすると、Xcode の [出力] ウィンドウにアクセストークンが表示されます。

![Xcode でアクセストークンが表示されている出力ウィンドウのスクリーンショット](./images/access-token-output.png)

## <a name="get-user-details"></a>ユーザーの詳細を取得する

このセクションでは、Microsoft Graph へのすべての呼び出しを保持するヘルパークラスを作成し、 `WelcomeViewController`を更新して、この新しいクラスを使用してログインしているユーザーを取得します。

1. **Authenticationmanager**を開き、次の関数を`AuthenticationManager`クラスに追加します。

    ```Swift
    public func getGraphAuthProvider() -> MSALAuthenticationProvider? {
        // Create an MSAL auth provider for use with the Graph client
        return MSALAuthenticationProvider(publicClientApplication: self.publicClient,
                                          andScopes: self.graphScopes)
    }
    ```

1. 「 **Graphtutorial. Swift**」という名前の**graphtutorial**プロジェクトに新しい**Swift ファイル**を作成します。 このファイルに次のコードを追加します。

    ```Swift
    import Foundation
    import MSGraphClientSDK
    import MSGraphClientModels

    class GraphManager {

        // Implement singleton pattern
        static let instance = GraphManager()

        private let client: MSHTTPClient?

        private init() {
            client = MSClientFactory.createHTTPClient(with: AuthenticationManager.instance.getGraphAuthProvider())
        }

        public func getMe(completion: @escaping(MSGraphUser?, Error?) -> Void) {
            // GET /me
            let meRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me")!)
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

1. **WelcomeViewController**を開き、次`import`のステートメントをファイルの先頭に追加します。

    ```Swift
    import MSGraphClientModels
    ```

1. 次のプロパティを`WelcomeViewController`クラスに追加します。

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. 既存`viewDidLoad`のを次のコードに置き換えます。

    ```Swift
    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.

        self.spinner.start(container: self)

        // Get the signed-in user
        self.userProfilePhoto.image = UIImage(imageLiteralResourceName: "DefaultUserPhoto")

        GraphManager.instance.getMe {
            (user: MSGraphUser?, error: Error?) in

            DispatchQueue.main.async {
                self.spinner.stop()

                guard let currentUser = user, error == nil else {
                    print("Error getting user: \(String(describing: error))")
                    return
                }

                // Set display name
                self.userDisplayName.text = currentUser.displayName ?? "Mysterious Stranger"
                self.userDisplayName.sizeToFit()

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = currentUser.mail ?? currentUser.userPrincipalName ?? ""
                self.userEmail.sizeToFit()
            }
        }
    }
    ```

変更を保存して今すぐアプリを再起動すると、サインイン後にユーザーの表示名と電子メールアドレスで UI が更新されます。
