<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="58b7d-101">まず、新しい Swift プロジェクトを作成します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-101">Begin by creating a new Swift project.</span></span>

1. <span data-ttu-id="58b7d-102">Xcode を開きます。</span><span class="sxs-lookup"><span data-stu-id="58b7d-102">Open Xcode.</span></span> <span data-ttu-id="58b7d-103">[ファイル] **メニューで** 、[新規]、次に [ **プロジェクト]** の順に **選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-103">On the **File** menu, select **New**, then **Project**.</span></span>
1. <span data-ttu-id="58b7d-104">アプリ テンプレートを **選択し** 、[次へ] を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-104">Choose the **App** template and select **Next**.</span></span>

    ![Xcode テンプレート選択ダイアログのスクリーンショット](images/xcode-select-template.png)

1. <span data-ttu-id="58b7d-106">製品名 **を Swift に設定** `GraphTutorial` し、 **言語** を Swift に **設定します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-106">Set the **Product Name** to `GraphTutorial` and the **Language** to **Swift**.</span></span>
1. <span data-ttu-id="58b7d-107">残りのフィールドに入力し、[次へ] を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-107">Fill in the remaining fields and select **Next**.</span></span>
1. <span data-ttu-id="58b7d-108">プロジェクトの場所を選択し、[作成] を選択 **します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-108">Choose a location for the project and select **Create**.</span></span>

## <a name="install-dependencies"></a><span data-ttu-id="58b7d-109">依存関係をインストールする</span><span class="sxs-lookup"><span data-stu-id="58b7d-109">Install dependencies</span></span>

<span data-ttu-id="58b7d-110">次に進む前に、後で使用する追加の依存関係をインストールします。</span><span class="sxs-lookup"><span data-stu-id="58b7d-110">Before moving on, install some additional dependencies that you will use later.</span></span>

- <span data-ttu-id="58b7d-111">Azure AD で認証を行う iOS 用[Microsoft Authentication Library (MSAL)。](https://github.com/AzureAD/microsoft-authentication-library-for-objc)</span><span class="sxs-lookup"><span data-stu-id="58b7d-111">[Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) for authenticating to with Azure AD.</span></span>
- <span data-ttu-id="58b7d-112">[Microsoft Graph への呼び出し](https://github.com/microsoftgraph/msgraph-sdk-objc) を行う目的 C 用 Microsoft Graph SDK。</span><span class="sxs-lookup"><span data-stu-id="58b7d-112">[Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="58b7d-113">[ユーザーやイベントなどの Microsoft Graph リソース](https://github.com/microsoftgraph/msgraph-sdk-objc-models) を表す、強力に型指定されたオブジェクト用の Microsoft Graph Models SDK for Objective C。</span><span class="sxs-lookup"><span data-stu-id="58b7d-113">[Microsoft Graph Models SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) for strongly-typed objects representing Microsoft Graph resources like users or events.</span></span>

1. <span data-ttu-id="58b7d-114">Xcode を終了します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-114">Quit Xcode.</span></span>
1. <span data-ttu-id="58b7d-115">ターミナルを開き、ディレクトリを **GraphTu読み込みプロジェクトの場所に変更** します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-115">Open Terminal and change the directory to the location of your **GraphTutorial** project.</span></span>
1. <span data-ttu-id="58b7d-116">Podfile を作成するには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-116">Run the following command to create a Podfile.</span></span>

    ```Shell
    pod init
    ```

1. <span data-ttu-id="58b7d-117">Podfile を開き、行の直後に次の行を追加 `use_frameworks!` します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-117">Open the Podfile and add the following lines just after the `use_frameworks!` line.</span></span>

    ```Ruby
    pod 'MSAL', '~> 1.1.13'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. <span data-ttu-id="58b7d-118">Podfile を保存し、次のコマンドを実行して依存関係をインストールします。</span><span class="sxs-lookup"><span data-stu-id="58b7d-118">Save the Podfile, then run the following command to install the dependencies.</span></span>

    ```Shell
    pod install
    ```

1. <span data-ttu-id="58b7d-119">コマンドが完了したら、Xcode で新しく作成した **GraphTutorial.xcworkspace を** 開きます。</span><span class="sxs-lookup"><span data-stu-id="58b7d-119">Once the command completes, open the newly created **GraphTutorial.xcworkspace** in Xcode.</span></span>

## <a name="design-the-app"></a><span data-ttu-id="58b7d-120">アプリを設計する</span><span class="sxs-lookup"><span data-stu-id="58b7d-120">Design the app</span></span>

<span data-ttu-id="58b7d-121">このセクションでは、サインイン ページ、タブ バー ナビゲーター、ウェルカム ページ、カレンダー ページなど、アプリのビューを作成します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-121">In this section you will create the views for the app: a sign in page, a tab bar navigator, a welcome page, and a calendar page.</span></span> <span data-ttu-id="58b7d-122">アクティビティ インジケーター オーバーレイも作成します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-122">You'll also create an activity indicator overlay.</span></span>

### <a name="create-sign-in-page"></a><span data-ttu-id="58b7d-123">サインイン ページを作成する</span><span class="sxs-lookup"><span data-stu-id="58b7d-123">Create sign in page</span></span>

1. <span data-ttu-id="58b7d-124">Xcode で **GraphTu読み込み** フォルダーを展開し **、ViewController.swift を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-124">Expand the **GraphTutorial** folder in Xcode, then select **ViewController.swift**.</span></span>
1. <span data-ttu-id="58b7d-125">ファイル検査 **で、** ファイル **の名前** を次に変更します `SignInViewController.swift` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-125">In the **File Inspector**, change the **Name** of the file to `SignInViewController.swift`.</span></span>

    ![ファイル検査のスクリーンショット](images/rename-file.png)

1. <span data-ttu-id="58b7d-127">**SignInViewController.swift を開き**、その内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="58b7d-127">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class SignInViewController: UIViewController {

        override func viewDidLoad() {
            super.viewDidLoad()
            // Do any additional setup after loading the view.
        }

        @IBAction func signIn() {
            self.performSegue(withIdentifier: "userSignedIn", sender: nil)
        }
    }
    ```

1. <span data-ttu-id="58b7d-128">**Main.storyboard を開きます**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-128">Open **Main.storyboard**.</span></span> <span data-ttu-id="58b7d-129">[ **コントローラー シーンの表示] を展開** し、[コントローラーの表示 **] を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-129">Expand **View Controller Scene**, then select **View Controller**.</span></span>

    ![ビュー コントローラーが選択されている Xcode のスクリーンショット](images/storyboard-select-view-controller.png)

1. <span data-ttu-id="58b7d-131">Identity Inspector **を選択し\*\*\*\*、Class** ドロップダウンを **SignInViewController に変更します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-131">Select the **Identity Inspector**, then change the **Class** dropdown to **SignInViewController**.</span></span>

    ![ID インスペクターのスクリーンショット](images/change-class.png)

1. <span data-ttu-id="58b7d-133">ライブラリを **選択し**、ボタンを **サインイン** ビュー **コントローラーにドラッグします**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-133">Select the **Library**, then drag a **Button** onto the **Sign In View Controller**.</span></span>

    ![Xcode のライブラリのスクリーンショット](images/add-button-to-view.png)

1. <span data-ttu-id="58b7d-135">ボタンを選択した後、属性インスペクタ **ーを** 選択し、ボタン **のタイトル** を次に変更します `Sign In` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-135">With the button selected, select the **Attributes Inspector** and change the **Title** of the button to `Sign In`.</span></span>

    ![Xcode の属性インスペクターの Title フィールドのスクリーンショット](images/set-button-title.png)

1. <span data-ttu-id="58b7d-137">ボタンを選択した後、ストーリーボードの下部にある [ **配置** ] ボタンを選択します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-137">With the button selected, select the **Align** button at the bottom of the storyboard.</span></span> <span data-ttu-id="58b7d-138">コンテナー内の **水平方向** と垂直方向の両方のコンテナー制約を選択し、その値を 0 のままにして、[2 の制約の追加 **] を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-138">Select both the **Horizontally in container** and **Vertically in container** constraints, leave their values as 0, then select **Add 2 constraints**.</span></span>

    ![Xcode の配置制約設定のスクリーンショット](images/add-alignment-constraints.png)

1. <span data-ttu-id="58b7d-140">サインイン ビュー **コントローラーを選択し、[** 接続インスペクター] **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-140">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="58b7d-141">[ **受信した操作]** の下で、入力されていない円をドラッグして **、[サインイン] ボタン** に移動します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-141">Under **Received Actions**, drag the unfilled circle next to **signIn** onto the button.</span></span> <span data-ttu-id="58b7d-142">ポップアップ **メニューの [内側に** タッチ] を選択します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-142">Select **Touch Up Inside** on the pop-up menu.</span></span>

    ![Xcode のボタンに SignIn アクションをドラッグするスクリーンショット](images/connect-sign-in-button.png)

### <a name="create-tab-bar"></a><span data-ttu-id="58b7d-144">タブ バーを作成する</span><span class="sxs-lookup"><span data-stu-id="58b7d-144">Create tab bar</span></span>

1. <span data-ttu-id="58b7d-145">ライブラリを **選択し**、タブ バー **コントローラーをストーリー** ボードにドラッグします。</span><span class="sxs-lookup"><span data-stu-id="58b7d-145">Select the **Library**, then drag a **Tab Bar Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="58b7d-146">サインイン ビュー **コントローラーを選択し、[** 接続インスペクター] **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-146">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="58b7d-147">**Triggered Segues の下** で、手動の横にある未入力の円をストーリーボードの **タブ バー** コントローラーにドラッグします。</span><span class="sxs-lookup"><span data-stu-id="58b7d-147">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Tab Bar Controller** on the storyboard.</span></span> <span data-ttu-id="58b7d-148">ポップアップ **メニューで [モーダルに** 表示] を選択します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-148">Select **Present Modally** in the pop-up menu.</span></span>

    ![Xcode の新しいタブ バー コントローラーに手動セグをドラッグするスクリーンショット](images/add-segue.png)

1. <span data-ttu-id="58b7d-150">追加したセグを選択し、属性インスペクターを **選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-150">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="58b7d-151">[識別子]**フィールドを** 設定 `userSignedIn` し、[プレゼンテーション] を [**全画面\*\*\*\*] に設定します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-151">Set the **Identifier** field to `userSignedIn`, and set **Presentation** to **Full Screen**.</span></span>

    ![Xcode の属性インスペクターの Identifier フィールドのスクリーンショット](images/set-segue-identifier.png)

1. <span data-ttu-id="58b7d-153">項目 **1 シーンを選択し**、[接続インスペクター] **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-153">Select the **Item 1 Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="58b7d-154">**Triggered Segues の下** で、手動の横にある入力されていない円をストーリーボードの **サインイン ビュー** コントローラーにドラッグします。</span><span class="sxs-lookup"><span data-stu-id="58b7d-154">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Sign In View Controller** on the storyboard.</span></span> <span data-ttu-id="58b7d-155">ポップアップ **メニューで [モーダルに** 表示] を選択します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-155">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="58b7d-156">追加したセグを選択し、属性インスペクターを **選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-156">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="58b7d-157">[識別子]**フィールドを** 設定 `userSignedOut` し、[プレゼンテーション] を [**全画面\*\*\*\*] に設定します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-157">Set the **Identifier** field to `userSignedOut`, and set **Presentation** to **Full Screen**.</span></span>

### <a name="create-welcome-page"></a><span data-ttu-id="58b7d-158">ウェルカム ページを作成する</span><span class="sxs-lookup"><span data-stu-id="58b7d-158">Create welcome page</span></span>

1. <span data-ttu-id="58b7d-159">**Assets.xcassets ファイルを選択** します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-159">Select the **Assets.xcassets** file.</span></span>
1. <span data-ttu-id="58b7d-160">[エディター **] メニューで** 、[新しいアセットの **追加] を** 選択し、[イメージ セット] **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-160">On the **Editor** menu, select **Add New Asset**, then **Image Set**.</span></span>
1. <span data-ttu-id="58b7d-161">新しい **画像アセットを選択** し、 **属性インスペクターを使** って名前を **設定** します `DefaultUserPhoto` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-161">Select the new **Image** asset and use the **Attribute Inspector** to set its **Name** to `DefaultUserPhoto`.</span></span>
1. <span data-ttu-id="58b7d-162">既定のユーザー プロファイル写真として使用する画像を追加します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-162">Add any image you like to serve as a default user profile photo.</span></span>

    ![Xcode のイメージ セット アセット ビューのスクリーンショット](images/add-default-user-photo.png)

1. <span data-ttu-id="58b7d-164">**GraphTu touchl** **フォルダーに新** しい Cocoa Touch Class ファイルを作成します `WelcomeViewController` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-164">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `WelcomeViewController`.</span></span> <span data-ttu-id="58b7d-165">フィールド **のサブクラスで UIViewController** **を選択** します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-165">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="58b7d-166">**WelcomeViewController.swift を開き、** その内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="58b7d-166">Open **WelcomeViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class WelcomeViewController: UIViewController {

        @IBOutlet var userProfilePhoto: UIImageView!
        @IBOutlet var userDisplayName: UILabel!
        @IBOutlet var userEmail: UILabel!

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.

            // TEMPORARY
            self.userProfilePhoto.image = UIImage(imageLiteralResourceName: "DefaultUserPhoto")
            self.userDisplayName.text = "Default User"
            self.userEmail.text = "default@contoso.com"
        }

        @IBAction func signOut() {
            self.performSegue(withIdentifier: "userSignedOut", sender: nil)
        }
    }
    ```

1. <span data-ttu-id="58b7d-167">**Main.storyboard を開きます**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-167">Open **Main.storyboard**.</span></span> <span data-ttu-id="58b7d-168">アイテム **1 シーンを選択し**、ID インスペクター **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-168">Select the **Item 1 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="58b7d-169">クラスの **値を** **WelcomeViewController に変更します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-169">Change the **Class** value to **WelcomeViewController**.</span></span>
1. <span data-ttu-id="58b7d-170">ライブラリを **使用して**、アイテム 1 シーンに **次の項目を追加します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-170">Using the **Library**, add the following items to the **Item 1 Scene**.</span></span>

    - <span data-ttu-id="58b7d-171">1 **つのイメージ ビュー**</span><span class="sxs-lookup"><span data-stu-id="58b7d-171">One **Image View**</span></span>
    - <span data-ttu-id="58b7d-172">2 **つのラベル**</span><span class="sxs-lookup"><span data-stu-id="58b7d-172">Two **Labels**</span></span>
    - <span data-ttu-id="58b7d-173">1 **つのボタン**</span><span class="sxs-lookup"><span data-stu-id="58b7d-173">One **Button**</span></span>
1. <span data-ttu-id="58b7d-174">Connections **Inspector を使用して、** 次の接続を行います。</span><span class="sxs-lookup"><span data-stu-id="58b7d-174">Using the **Connections Inspector**, make the following connections.</span></span>

    - <span data-ttu-id="58b7d-175">**userDisplayName の出口を** 最初のラベルにリンクします。</span><span class="sxs-lookup"><span data-stu-id="58b7d-175">Link the **userDisplayName** outlet to the first label.</span></span>
    - <span data-ttu-id="58b7d-176">**userEmail の出口を** 2 番目のラベルにリンクします。</span><span class="sxs-lookup"><span data-stu-id="58b7d-176">Link the **userEmail** outlet to the second label.</span></span>
    - <span data-ttu-id="58b7d-177">**userProfilePhoto の出口を** イメージ ビューにリンクします。</span><span class="sxs-lookup"><span data-stu-id="58b7d-177">Link the **userProfilePhoto** outlet to the image view.</span></span>
    - <span data-ttu-id="58b7d-178">signOut **受信アクションを** ボタンの Touch **Up Inside にリンクします**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-178">Link the **signOut** received action to the button's **Touch Up Inside**.</span></span>

1. <span data-ttu-id="58b7d-179">イメージ ビューを選択し、サイズ検査を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-179">Select the image view, then select the **Size Inspector**.</span></span>
1. <span data-ttu-id="58b7d-180">幅と **高** さを 196 に設定します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-180">Set the **Width** and **Height** to 196.</span></span>
1. <span data-ttu-id="58b7d-181">[配置 **] ボタン** を使用して、 **コンテナー内で水平方向に** 制約を追加し、値を 0 に設定します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-181">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="58b7d-182">[新しい **制約の追加]** ボタン([整列] ボタンの横) を使用して、次の制約を追加します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-182">Use the **Add New Constraints** button (next to the **Align** button) to add the following constraints:</span></span>

    - <span data-ttu-id="58b7d-183">上揃え: セーフ エリア、値: 0</span><span class="sxs-lookup"><span data-stu-id="58b7d-183">Align Top to: Safe Area, value: 0</span></span>
    - <span data-ttu-id="58b7d-184">[下のスペース]: ユーザー表示名、値: Standard</span><span class="sxs-lookup"><span data-stu-id="58b7d-184">Bottom Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="58b7d-185">高さ、値: 196</span><span class="sxs-lookup"><span data-stu-id="58b7d-185">Height, value: 196</span></span>
    - <span data-ttu-id="58b7d-186">幅、値: 196</span><span class="sxs-lookup"><span data-stu-id="58b7d-186">Width, value: 196</span></span>

    ![Xcode の新しい制約設定のスクリーンショット](./images/add-new-constraints.png)

1. <span data-ttu-id="58b7d-188">1 つ目のラベルを選択し、[配置] ボタンを使用して、コンテナー内の水平方向 **の** 制約を値 0 で追加します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-188">Select the first label, then use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="58b7d-189">[新しい **制約の追加] ボタンを使用** して、次の制約を追加します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-189">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="58b7d-190">Top Space to: User Profile Photo, value: Standard</span><span class="sxs-lookup"><span data-stu-id="58b7d-190">Top Space to: User Profile Photo, value: Standard</span></span>
    - <span data-ttu-id="58b7d-191">[Bottom Space to]: User Email, value: Standard</span><span class="sxs-lookup"><span data-stu-id="58b7d-191">Bottom Space to: User Email, value: Standard</span></span>

1. <span data-ttu-id="58b7d-192">2 番目のラベルを選択し、属性インスペクター **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-192">Select the second label, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="58b7d-193">色を **濃い** 灰色 **に変更** し、 **フォント** をシステム **12.0 に変更します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-193">Change the **Color** to **Dark Gray Color**, and change the **Font** to **System 12.0**.</span></span>
1. <span data-ttu-id="58b7d-194">[配置 **] ボタン** を使用して、 **コンテナー内で水平方向に** 制約を追加し、値を 0 に設定します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-194">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="58b7d-195">[新しい **制約の追加] ボタンを使用** して、次の制約を追加します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-195">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="58b7d-196">[Top Space to]: User Display Name,value: Standard</span><span class="sxs-lookup"><span data-stu-id="58b7d-196">Top Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="58b7d-197">下のスペース: サインアウト、値: 14</span><span class="sxs-lookup"><span data-stu-id="58b7d-197">Bottom Space to: Sign Out, value: 14</span></span>

1. <span data-ttu-id="58b7d-198">ボタンを選択し、属性インスペクタ **ーを選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-198">Select the button, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="58b7d-199">Title を **次に変更** します `Sign Out` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-199">Change the **Title** to `Sign Out`.</span></span>
1. <span data-ttu-id="58b7d-200">[配置 **] ボタン** を使用して、 **コンテナー内で水平方向に** 制約を追加し、値を 0 に設定します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-200">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="58b7d-201">[新しい **制約の追加] ボタンを使用** して、次の制約を追加します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-201">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="58b7d-202">Top Space to: User Email, value: 14</span><span class="sxs-lookup"><span data-stu-id="58b7d-202">Top Space to: User Email, value: 14</span></span>

1. <span data-ttu-id="58b7d-203">シーンの下部にあるタブ バー項目を選択し、[属性インスペクター] を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-203">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="58b7d-204">Title を **次に変更** します `Me` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-204">Change the **Title** to `Me`.</span></span>

<span data-ttu-id="58b7d-205">完了すると、ようこそシーンは次のように表示されます。</span><span class="sxs-lookup"><span data-stu-id="58b7d-205">The welcome scene should look similar to this once you're done.</span></span>

![ようこそシーンのレイアウトのスクリーンショット](images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a><span data-ttu-id="58b7d-207">予定表ページを作成する</span><span class="sxs-lookup"><span data-stu-id="58b7d-207">Create calendar page</span></span>

1. <span data-ttu-id="58b7d-208">**GraphTu touchl** **フォルダーに新** しい Cocoa Touch Class ファイルを作成します `CalendarViewController` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-208">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `CalendarViewController`.</span></span> <span data-ttu-id="58b7d-209">フィールド **のサブクラスで UIViewController** **を選択** します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-209">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="58b7d-210">**CalendarViewController.swift を開き、** 内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="58b7d-210">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class CalendarViewController: UIViewController {

        @IBOutlet var calendarJSON: UITextView!

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.

            // TEMPORARY
            calendarJSON.text = "Calendar"
            calendarJSON.sizeToFit()
        }
    }
    ```

1. <span data-ttu-id="58b7d-211">**Main.storyboard を開きます**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-211">Open **Main.storyboard**.</span></span> <span data-ttu-id="58b7d-212">アイテム **2 シーンを選択し**、ID インスペクター **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-212">Select the **Item 2 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="58b7d-213">クラスの **値を** **CalendarViewController に変更します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-213">Change the **Class** value to **CalendarViewController**.</span></span>
1. <span data-ttu-id="58b7d-214">ライブラリを **使用して**、アイテム 2 **シーンに\*\*\*\*テキスト ビューを追加します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-214">Using the **Library**, add a **Text View** to the **Item 2 Scene**.</span></span>
1. <span data-ttu-id="58b7d-215">追加したテキスト ビューを選択します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-215">Select the text view you just added.</span></span> <span data-ttu-id="58b7d-216">[エディター] **メニューで** 、[埋め込み **]、次** に [スクロール ビュー] **の順に選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-216">On the **Editor** menu, choose **Embed In**, then **Scroll View**.</span></span>
1. <span data-ttu-id="58b7d-217">スクロール ビューとテキスト ビューのサイズを変更して、画面全体を表示します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-217">Resize the scroll view and text view to take up the entire screen.</span></span>
1. <span data-ttu-id="58b7d-218">Connections **Inspector を使用して\*\*\*\*、calendarJSON** の出口をテキスト ビューに接続します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-218">Using the **Connections Inspector**, connect the **calendarJSON** outlet to the text view.</span></span>
1. <span data-ttu-id="58b7d-219">シーンの下部にあるタブ バー項目を選択し、[属性インスペクター] を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-219">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="58b7d-220">Title を **次に変更** します `Calendar` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-220">Change the **Title** to `Calendar`.</span></span>
1. <span data-ttu-id="58b7d-221">[エディター **] メニューの**[自動レイアウトの問題の解決] を選択し、予定表ビュー コントローラーのすべてのビューの下にある [不足している制約の追加]**を選択します**。</span><span class="sxs-lookup"><span data-stu-id="58b7d-221">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Calendar View Controller**.</span></span>

<span data-ttu-id="58b7d-222">完了すると、予定表シーンは次のように表示されます。</span><span class="sxs-lookup"><span data-stu-id="58b7d-222">The calendar scene should look similar to this once you're done.</span></span>

![カレンダー シーン レイアウトのスクリーンショット](images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a><span data-ttu-id="58b7d-224">アクティビティ インジケーターを作成する</span><span class="sxs-lookup"><span data-stu-id="58b7d-224">Create activity indicator</span></span>

1. <span data-ttu-id="58b7d-225">**GraphTu touchl** **フォルダーに新** しい Cocoa Touch Class ファイルを作成します `SpinnerViewController` 。</span><span class="sxs-lookup"><span data-stu-id="58b7d-225">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `SpinnerViewController`.</span></span> <span data-ttu-id="58b7d-226">フィールド **のサブクラスで UIViewController** **を選択** します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-226">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="58b7d-227">**SpinnerViewController.swift を開き**、内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="58b7d-227">Open **SpinnerViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SpinnerViewController.swift" id="SpinnerSnippet":::

## <a name="test-the-app"></a><span data-ttu-id="58b7d-228">アプリのテスト</span><span class="sxs-lookup"><span data-stu-id="58b7d-228">Test the app</span></span>

<span data-ttu-id="58b7d-229">変更内容を保存し、アプリを起動します。</span><span class="sxs-lookup"><span data-stu-id="58b7d-229">Save your changes and launch the app.</span></span> <span data-ttu-id="58b7d-230">[サインイン] ボタンと [サインアウト]ボタンとタブ バーを使用して、画面間を移動できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="58b7d-230">You should be able to move between the screens using the **Sign In** and **Sign Out** buttons and the tab bar.</span></span>

![アプリケーションのスクリーンショット](images/app-screens.png)
