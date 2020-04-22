<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="0aaf7-101">この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="0aaf7-102">これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="0aaf7-103">これを行うには、 [iOS 用 Microsoft Authentication Library (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-objc)をアプリケーションに統合します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="0aaf7-104">**Authsettings. plist**という名前の**graphtutorial**プロジェクトに、新しい**プロパティリスト**ファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="0aaf7-105">次の項目を**ルート**ディクショナリのファイルに追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="0aaf7-106">キー</span><span class="sxs-lookup"><span data-stu-id="0aaf7-106">Key</span></span> | <span data-ttu-id="0aaf7-107">型</span><span class="sxs-lookup"><span data-stu-id="0aaf7-107">Type</span></span> | <span data-ttu-id="0aaf7-108">値</span><span class="sxs-lookup"><span data-stu-id="0aaf7-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="0aaf7-109">String</span><span class="sxs-lookup"><span data-stu-id="0aaf7-109">String</span></span> | <span data-ttu-id="0aaf7-110">Azure portal からのアプリケーション ID</span><span class="sxs-lookup"><span data-stu-id="0aaf7-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="0aaf7-111">配列</span><span class="sxs-lookup"><span data-stu-id="0aaf7-111">Array</span></span> | <span data-ttu-id="0aaf7-112">2つの文字列`User.Read`値:`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="0aaf7-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Xcode の AuthSettings plist ファイルのスクリーンショット](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="0aaf7-114">Git などのソース管理を使用している場合は、アプリ ID が誤ってリークしないように、ソース管理から**Authsettings の設定**ファイルを除外することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="0aaf7-115">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="0aaf7-115">Implement sign-in</span></span>

<span data-ttu-id="0aaf7-116">このセクションでは、MSAL のプロジェクトを構成し、認証マネージャークラスを作成し、サインインしてサインアウトするためにアプリを更新します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="0aaf7-117">MSAL のプロジェクトを構成する</span><span class="sxs-lookup"><span data-stu-id="0aaf7-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="0aaf7-118">プロジェクトの機能に新しいキーチェーングループを追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="0aaf7-119">**Graphtutorial**プロジェクトを選択し、 **& 機能に署名**します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="0aaf7-120">[ **+ 機能**] を選択し、[**キーチェーン共有**] をダブルクリックします。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="0aaf7-121">値`com.microsoft.adalcache`を持つキーチェーングループを追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="0aaf7-122">[コントロール] をクリックし、[**名前を付けて開く**]、[**ソースコード**] の順に選択し**ます。**</span><span class="sxs-lookup"><span data-stu-id="0aaf7-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="0aaf7-123">`<dict>`要素内に次のように追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="0aaf7-124">**Appdelegate**を開き、ファイルの先頭に次の import ステートメントを追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-124">Open **AppDelegate.swift** and add the following import statement at the top of the file.</span></span>

    ```Swift
    import MSAL
    ```

1. <span data-ttu-id="0aaf7-125">次の関数を `AppDelegate` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="0aaf7-126">認証マネージャーを作成する</span><span class="sxs-lookup"><span data-stu-id="0aaf7-126">Create authentication manager</span></span>

1. <span data-ttu-id="0aaf7-127">「 **Authenticationmanager. Swift**」という名前の**graphtutorial**プロジェクトに新しい**Swift ファイル**を作成します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-127">Create a new **Swift File** in the **GraphTutorial** project named **AuthenticationManager.swift**.</span></span> <span data-ttu-id="0aaf7-128">このファイルに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-128">Add the following code to the file.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="0aaf7-129">サインインとサインアウトを追加する</span><span class="sxs-lookup"><span data-stu-id="0aaf7-129">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="0aaf7-130">**SignInViewController**を開き、そのコンテンツを次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-130">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. <span data-ttu-id="0aaf7-131">**WelcomeViewController**を開き、既存`signOut`の関数を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-131">Open **WelcomeViewController.swift** and replace the existing `signOut` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. <span data-ttu-id="0aaf7-132">変更を保存し、シミュレータでアプリケーションを再起動します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-132">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="0aaf7-133">アプリにサインインすると、Xcode の [出力] ウィンドウにアクセストークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-133">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Xcode でアクセストークンが表示されている出力ウィンドウのスクリーンショット](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="0aaf7-135">ユーザーの詳細情報を取得する</span><span class="sxs-lookup"><span data-stu-id="0aaf7-135">Get user details</span></span>

<span data-ttu-id="0aaf7-136">このセクションでは、Microsoft Graph へのすべての呼び出しを保持するヘルパークラスを作成し、 `WelcomeViewController`を更新して、この新しいクラスを使用してログインしているユーザーを取得します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-136">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="0aaf7-137">「 **Graphtutorial. Swift**」という名前の**graphtutorial**プロジェクトに新しい**Swift ファイル**を作成します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-137">Create a new **Swift File** in the **GraphTutorial** project named **GraphManager.swift**.</span></span> <span data-ttu-id="0aaf7-138">このファイルに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-138">Add the following code to the file.</span></span>

    ```Swift
    import Foundation
    import MSGraphClientSDK
    import MSGraphClientModels

    class GraphManager {

        // Implement singleton pattern
        static let instance = GraphManager()

        private let client: MSHTTPClient?

        private init() {
            client = MSClientFactory.createHTTPClient(with: AuthenticationManager.instance)
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

1. <span data-ttu-id="0aaf7-139">**WelcomeViewController**を開き、次`import`のステートメントをファイルの先頭に追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-139">Open **WelcomeViewController.swift** and add the following `import` statement at the top of the file.</span></span>

    ```Swift
    import MSGraphClientModels
    ```

1. <span data-ttu-id="0aaf7-140">次のプロパティを `WelcomeViewController` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-140">Add the following property to the `WelcomeViewController` class.</span></span>

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. <span data-ttu-id="0aaf7-141">既存`viewDidLoad`のを次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-141">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

<span data-ttu-id="0aaf7-142">変更を保存して今すぐアプリを再起動すると、サインイン後にユーザーの表示名と電子メールアドレスで UI が更新されます。</span><span class="sxs-lookup"><span data-stu-id="0aaf7-142">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
