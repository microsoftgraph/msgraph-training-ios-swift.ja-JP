<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="94b16-101">この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。</span><span class="sxs-lookup"><span data-stu-id="94b16-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="94b16-102">これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="94b16-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="94b16-103">これを行うには、 [iOS 用 Microsoft Authentication Library (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-objc)をアプリケーションに統合します。</span><span class="sxs-lookup"><span data-stu-id="94b16-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="94b16-104">**Authsettings. plist**という名前の**graphtutorial**プロジェクトに、新しい**プロパティリスト**ファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="94b16-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="94b16-105">次の項目を**ルート**ディクショナリのファイルに追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="94b16-106">Key</span><span class="sxs-lookup"><span data-stu-id="94b16-106">Key</span></span> | <span data-ttu-id="94b16-107">型</span><span class="sxs-lookup"><span data-stu-id="94b16-107">Type</span></span> | <span data-ttu-id="94b16-108">値</span><span class="sxs-lookup"><span data-stu-id="94b16-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="94b16-109">String</span><span class="sxs-lookup"><span data-stu-id="94b16-109">String</span></span> | <span data-ttu-id="94b16-110">Azure portal からのアプリケーション ID</span><span class="sxs-lookup"><span data-stu-id="94b16-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="94b16-111">配列</span><span class="sxs-lookup"><span data-stu-id="94b16-111">Array</span></span> | <span data-ttu-id="94b16-112">2つの文字列`User.Read`値:`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="94b16-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Xcode の AuthSettings plist ファイルのスクリーンショット](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="94b16-114">Git などのソース管理を使用している場合は、アプリ ID が誤ってリークしないように、ソース管理から**Authsettings の設定**ファイルを除外することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="94b16-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="94b16-115">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="94b16-115">Implement sign-in</span></span>

<span data-ttu-id="94b16-116">このセクションでは、MSAL のプロジェクトを構成し、認証マネージャークラスを作成し、サインインしてサインアウトするためにアプリを更新します。</span><span class="sxs-lookup"><span data-stu-id="94b16-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="94b16-117">MSAL のプロジェクトを構成する</span><span class="sxs-lookup"><span data-stu-id="94b16-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="94b16-118">[コントロール] をクリックし、[**名前を付けて開く**]、[**ソースコード**] の順に選択し**ます。**</span><span class="sxs-lookup"><span data-stu-id="94b16-118">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="94b16-119">`<dict>`要素内に次のように追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-119">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="94b16-120">**Appdelegate**を開き、ファイルの先頭に次の import ステートメントを追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-120">Open **AppDelegate.swift** and add the following import statement at the top of the file.</span></span>

    ```Swift
    import MSAL
    ```

1. <span data-ttu-id="94b16-121">次の関数を `AppDelegate` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-121">Add the following function to the `AppDelegate` class.</span></span>

    ```Swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        guard let sourceApplication = options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String else {
            return false
        }

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApplication)
    }
    ```

### <a name="create-authentication-manager"></a><span data-ttu-id="94b16-122">認証マネージャーを作成する</span><span class="sxs-lookup"><span data-stu-id="94b16-122">Create authentication manager</span></span>

1. <span data-ttu-id="94b16-123">「 **Authenticationmanager. Swift**」という名前の**graphtutorial**プロジェクトに新しい**Swift ファイル**を作成します。</span><span class="sxs-lookup"><span data-stu-id="94b16-123">Create a new **Swift File** in the **GraphTutorial** project named **AuthenticationManager.swift**.</span></span> <span data-ttu-id="94b16-124">このファイルに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-124">Add the following code to the file.</span></span>

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

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="94b16-125">サインインとサインアウトを追加する</span><span class="sxs-lookup"><span data-stu-id="94b16-125">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="94b16-126">**SignInViewController**を開き、そのコンテンツを次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="94b16-126">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="94b16-127">**WelcomeViewController**を開き、既存`signOut`の関数を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="94b16-127">Open **WelcomeViewController.swift** and replace the existing `signOut` function with the following.</span></span>

    ```Swift
    @IBAction func signOut() {
        AuthenticationManager.instance.signOut()
        self.performSegue(withIdentifier: "userSignedOut", sender: nil)
    }
    ```

1. <span data-ttu-id="94b16-128">変更を保存し、シミュレータでアプリケーションを再起動します。</span><span class="sxs-lookup"><span data-stu-id="94b16-128">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="94b16-129">アプリにサインインすると、Xcode の [出力] ウィンドウにアクセストークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="94b16-129">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Xcode でアクセストークンが表示されている出力ウィンドウのスクリーンショット](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="94b16-131">ユーザーの詳細を取得する</span><span class="sxs-lookup"><span data-stu-id="94b16-131">Get user details</span></span>

<span data-ttu-id="94b16-132">このセクションでは、Microsoft Graph へのすべての呼び出しを保持するヘルパークラスを作成し、 `WelcomeViewController`を更新して、この新しいクラスを使用してログインしているユーザーを取得します。</span><span class="sxs-lookup"><span data-stu-id="94b16-132">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="94b16-133">**Authenticationmanager**を開き、次の関数を`AuthenticationManager`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-133">Open **AuthenticationManager.swift** and add the following function to the `AuthenticationManager` class.</span></span>

    ```Swift
    public func getGraphAuthProvider() -> MSALAuthenticationProvider? {
        // Create an MSAL auth provider for use with the Graph client
        return MSALAuthenticationProvider(publicClientApplication: self.publicClient,
                                          andScopes: self.graphScopes)
    }
    ```

1. <span data-ttu-id="94b16-134">「 **Graphtutorial. Swift**」という名前の**graphtutorial**プロジェクトに新しい**Swift ファイル**を作成します。</span><span class="sxs-lookup"><span data-stu-id="94b16-134">Create a new **Swift File** in the **GraphTutorial** project named **GraphManager.swift**.</span></span> <span data-ttu-id="94b16-135">このファイルに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-135">Add the following code to the file.</span></span>

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

1. <span data-ttu-id="94b16-136">**WelcomeViewController**を開き、次`import`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-136">Open **WelcomeViewController.swift** and add the following `import` statement at the top of the file.</span></span>

    ```Swift
    import MSGraphClientModels
    ```

1. <span data-ttu-id="94b16-137">次のプロパティを`WelcomeViewController`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="94b16-137">Add the following property to the `WelcomeViewController` class.</span></span>

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. <span data-ttu-id="94b16-138">既存`viewDidLoad`のを次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="94b16-138">Replace the existing `viewDidLoad` with the following code.</span></span>

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

<span data-ttu-id="94b16-139">変更を保存して今すぐアプリを再起動すると、サインイン後にユーザーの表示名と電子メールアドレスで UI が更新されます。</span><span class="sxs-lookup"><span data-stu-id="94b16-139">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>