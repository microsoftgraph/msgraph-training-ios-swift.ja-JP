<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9778f-101">この演習では、前の演習のアプリケーションを拡張して、Azure AD での認証をサポートします。</span><span class="sxs-lookup"><span data-stu-id="9778f-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="9778f-102">これは、Microsoft Graph を呼び出すのに必要な OAuth アクセス トークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="9778f-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="9778f-103">これを行うには [、iOS 用 Microsoft Authentication Library (MSAL) を](https://github.com/AzureAD/microsoft-authentication-library-for-objc) アプリケーションに統合します。</span><span class="sxs-lookup"><span data-stu-id="9778f-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="9778f-104">**AuthSettings.plist** という名前の **GraphTu graphl** プロジェクトに新しいプロパティ リスト ファイルを作成します。 </span><span class="sxs-lookup"><span data-stu-id="9778f-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="9778f-105">ルート ディクショナリ内のファイルに次の項目 **を** 追加します。</span><span class="sxs-lookup"><span data-stu-id="9778f-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="9778f-106">キー</span><span class="sxs-lookup"><span data-stu-id="9778f-106">Key</span></span> | <span data-ttu-id="9778f-107">型</span><span class="sxs-lookup"><span data-stu-id="9778f-107">Type</span></span> | <span data-ttu-id="9778f-108">値</span><span class="sxs-lookup"><span data-stu-id="9778f-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="9778f-109">String</span><span class="sxs-lookup"><span data-stu-id="9778f-109">String</span></span> | <span data-ttu-id="9778f-110">Azure Portal からのアプリケーション ID</span><span class="sxs-lookup"><span data-stu-id="9778f-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="9778f-111">配列</span><span class="sxs-lookup"><span data-stu-id="9778f-111">Array</span></span> | <span data-ttu-id="9778f-112">3 つの文字列値: `User.Read` `MailboxSettings.Read` 、 、 `Calendars.ReadWrite`</span><span class="sxs-lookup"><span data-stu-id="9778f-112">Three String values: `User.Read`, `MailboxSettings.Read`, and `Calendars.ReadWrite`</span></span> |

    ![Xcode の AuthSettings.plist ファイルのスクリーンショット](images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="9778f-114">git などのソース管理を使用している場合は **、AuthSettings.plist** ファイルをソース管理から除外して、アプリ ID が誤って漏洩しないようにする良い時期です。</span><span class="sxs-lookup"><span data-stu-id="9778f-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="9778f-115">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="9778f-115">Implement sign-in</span></span>

<span data-ttu-id="9778f-116">このセクションでは、MSAL のプロジェクトを構成し、認証マネージャー クラスを作成し、サインインしてサインアウトするアプリを更新します。</span><span class="sxs-lookup"><span data-stu-id="9778f-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="9778f-117">MSAL 用にプロジェクトを構成する</span><span class="sxs-lookup"><span data-stu-id="9778f-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="9778f-118">プロジェクトの機能に新しいキーチェーン グループを追加します。</span><span class="sxs-lookup"><span data-stu-id="9778f-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="9778f-119">**GraphTu読み込みプロジェクトを** 選択し、次に **[&機能] を選択します**。</span><span class="sxs-lookup"><span data-stu-id="9778f-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="9778f-120">[+ **機能] を** 選択し、[キーチェーンの共有] **をダブルクリックします**。</span><span class="sxs-lookup"><span data-stu-id="9778f-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="9778f-121">値を持つキーチェーン グループを追加します `com.microsoft.adalcache` 。</span><span class="sxs-lookup"><span data-stu-id="9778f-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="9778f-122">**[Info.plist] をクリックし、[** ファイル名を指定して **開く]、次** に [ソース コード]**の順に選択します**。</span><span class="sxs-lookup"><span data-stu-id="9778f-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="9778f-123">要素内に次を追加 `<dict>` します。</span><span class="sxs-lookup"><span data-stu-id="9778f-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="9778f-124">**AppDelegate.swift を開** き、ファイルの上部に次の import ステートメントを追加します。</span><span class="sxs-lookup"><span data-stu-id="9778f-124">Open **AppDelegate.swift** and add the following import statement at the top of the file.</span></span>

    ```Swift
    import MSAL
    ```

1. <span data-ttu-id="9778f-125">次の関数を `AppDelegate` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="9778f-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="9778f-126">認証マネージャーを作成する</span><span class="sxs-lookup"><span data-stu-id="9778f-126">Create authentication manager</span></span>

1. <span data-ttu-id="9778f-127">AuthenticationManager.swift **という名前の** **GraphTu graphl** プロジェクトに新しい **Swift ファイルを作成します**。</span><span class="sxs-lookup"><span data-stu-id="9778f-127">Create a new **Swift File** in the **GraphTutorial** project named **AuthenticationManager.swift**.</span></span> <span data-ttu-id="9778f-128">次のコードをファイルに追加します。</span><span class="sxs-lookup"><span data-stu-id="9778f-128">Add the following code to the file.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="9778f-129">サインインとサインアウトを追加する</span><span class="sxs-lookup"><span data-stu-id="9778f-129">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="9778f-130">**SignInViewController.swift を開き**、その内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="9778f-130">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. <span data-ttu-id="9778f-131">**WelcomeViewController.swift を開き**、既存の `signOut` 関数を次の関数に置き換える。</span><span class="sxs-lookup"><span data-stu-id="9778f-131">Open **WelcomeViewController.swift** and replace the existing `signOut` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. <span data-ttu-id="9778f-132">変更を保存し、シミュレーターでアプリケーションを再起動します。</span><span class="sxs-lookup"><span data-stu-id="9778f-132">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="9778f-133">アプリにサインインすると、Xcode の出力ウィンドウにアクセス トークンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="9778f-133">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![アクセス トークンを示す Xcode の出力ウィンドウのスクリーンショット](images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="9778f-135">ユーザーの詳細情報を取得する</span><span class="sxs-lookup"><span data-stu-id="9778f-135">Get user details</span></span>

<span data-ttu-id="9778f-136">このセクションでは、Microsoft Graph へのすべての呼び出しを保持するヘルパー クラスを作成し、この新しいクラスを使用してログイン ユーザーを取得するために更新 `WelcomeViewController` します。</span><span class="sxs-lookup"><span data-stu-id="9778f-136">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="9778f-137">GraphManager.swift **という名前の** **GraphTu graphl** プロジェクトに新しい **Swift ファイルを作成します**。</span><span class="sxs-lookup"><span data-stu-id="9778f-137">Create a new **Swift File** in the **GraphTutorial** project named **GraphManager.swift**.</span></span> <span data-ttu-id="9778f-138">次のコードをファイルに追加します。</span><span class="sxs-lookup"><span data-stu-id="9778f-138">Add the following code to the file.</span></span>

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

1. <span data-ttu-id="9778f-139">**WelcomeViewController.swift を** 開き、ファイルの上部に `import` 次のステートメントを追加します。</span><span class="sxs-lookup"><span data-stu-id="9778f-139">Open **WelcomeViewController.swift** and add the following `import` statement at the top of the file.</span></span>

    ```Swift
    import MSGraphClientModels
    ```

1. <span data-ttu-id="9778f-140">次のプロパティを `WelcomeViewController` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="9778f-140">Add the following property to the `WelcomeViewController` class.</span></span>

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. <span data-ttu-id="9778f-141">既存のコードを次 `viewDidLoad` のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="9778f-141">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

<span data-ttu-id="9778f-142">変更を保存してアプリを今すぐ再起動すると、サインイン後にユーザーの表示名とメール アドレスで UI が更新されます。</span><span class="sxs-lookup"><span data-stu-id="9778f-142">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
