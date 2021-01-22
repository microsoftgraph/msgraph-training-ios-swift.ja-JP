<!-- markdownlint-disable MD002 MD041 -->

まず、新しい Swift プロジェクトを作成します。

1. Xcode を開きます。 [ファイル] **メニューで** 、[新規]、次に [ **プロジェクト]** の順に **選択します**。
1. アプリ テンプレートを **選択し** 、[次へ] を **選択します**。

    ![Xcode テンプレート選択ダイアログのスクリーンショット](images/xcode-select-template.png)

1. 製品名 **を Swift に設定** `GraphTutorial` し、 **言語** を Swift に **設定します**。
1. 残りのフィールドに入力し、[次へ] を **選択します**。
1. プロジェクトの場所を選択し、[作成] を選択 **します**。

## <a name="install-dependencies"></a>依存関係をインストールする

次に進む前に、後で使用する追加の依存関係をインストールします。

- Azure AD で認証を行う iOS 用[Microsoft Authentication Library (MSAL)。](https://github.com/AzureAD/microsoft-authentication-library-for-objc)
- [Microsoft Graph への呼び出し](https://github.com/microsoftgraph/msgraph-sdk-objc) を行う目的 C 用 Microsoft Graph SDK。
- [ユーザーやイベントなどの Microsoft Graph リソース](https://github.com/microsoftgraph/msgraph-sdk-objc-models) を表す、強力に型指定されたオブジェクト用の Microsoft Graph Models SDK for Objective C。

1. Xcode を終了します。
1. ターミナルを開き、ディレクトリを **GraphTu読み込みプロジェクトの場所に変更** します。
1. Podfile を作成するには、次のコマンドを実行します。

    ```Shell
    pod init
    ```

1. Podfile を開き、行の直後に次の行を追加 `use_frameworks!` します。

    ```Ruby
    pod 'MSAL', '~> 1.1.13'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. Podfile を保存し、次のコマンドを実行して依存関係をインストールします。

    ```Shell
    pod install
    ```

1. コマンドが完了したら、Xcode で新しく作成した **GraphTutorial.xcworkspace を** 開きます。

## <a name="design-the-app"></a>アプリを設計する

このセクションでは、サインイン ページ、タブ バー ナビゲーター、ウェルカム ページ、カレンダー ページなど、アプリのビューを作成します。 アクティビティ インジケーター オーバーレイも作成します。

### <a name="create-sign-in-page"></a>サインイン ページを作成する

1. Xcode で **GraphTu読み込み** フォルダーを展開し **、ViewController.swift を選択します**。
1. ファイル検査 **で、** ファイル **の名前** を次に変更します `SignInViewController.swift` 。

    ![ファイル検査のスクリーンショット](images/rename-file.png)

1. **SignInViewController.swift を開き**、その内容を次のコードに置き換えます。

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

1. **Main.storyboard を開きます**。 [ **コントローラー シーンの表示] を展開** し、[コントローラーの表示 **] を選択します**。

    ![ビュー コントローラーが選択されている Xcode のスクリーンショット](images/storyboard-select-view-controller.png)

1. Identity Inspector **を選択し****、Class** ドロップダウンを **SignInViewController に変更します**。

    ![ID インスペクターのスクリーンショット](images/change-class.png)

1. ライブラリを **選択し**、ボタンを **サインイン** ビュー **コントローラーにドラッグします**。

    ![Xcode のライブラリのスクリーンショット](images/add-button-to-view.png)

1. ボタンを選択した後、属性インスペクタ **ーを** 選択し、ボタン **のタイトル** を次に変更します `Sign In` 。

    ![Xcode の属性インスペクターの Title フィールドのスクリーンショット](images/set-button-title.png)

1. ボタンを選択した後、ストーリーボードの下部にある [ **配置** ] ボタンを選択します。 コンテナー内の **水平方向** と垂直方向の両方のコンテナー制約を選択し、その値を 0 のままにして、[2 の制約の追加 **] を選択します**。

    ![Xcode の配置制約設定のスクリーンショット](images/add-alignment-constraints.png)

1. サインイン ビュー **コントローラーを選択し、[** 接続インスペクター] **を選択します**。
1. [ **受信した操作]** の下で、入力されていない円をドラッグして **、[サインイン] ボタン** に移動します。 ポップアップ **メニューの [内側に** タッチ] を選択します。

    ![Xcode のボタンに SignIn アクションをドラッグするスクリーンショット](images/connect-sign-in-button.png)

### <a name="create-tab-bar"></a>タブ バーを作成する

1. ライブラリを **選択し**、タブ バー **コントローラーをストーリー** ボードにドラッグします。
1. サインイン ビュー **コントローラーを選択し、[** 接続インスペクター] **を選択します**。
1. **Triggered Segues の下** で、手動の横にある未入力の円をストーリーボードの **タブ バー** コントローラーにドラッグします。 ポップアップ **メニューで [モーダルに** 表示] を選択します。

    ![Xcode の新しいタブ バー コントローラーに手動セグをドラッグするスクリーンショット](images/add-segue.png)

1. 追加したセグを選択し、属性インスペクターを **選択します**。 [識別子]**フィールドを** 設定 `userSignedIn` し、[プレゼンテーション] を [**全画面****] に設定します**。

    ![Xcode の属性インスペクターの Identifier フィールドのスクリーンショット](images/set-segue-identifier.png)

1. 項目 **1 シーンを選択し**、[接続インスペクター] **を選択します**。
1. **Triggered Segues の下** で、手動の横にある入力されていない円をストーリーボードの **サインイン ビュー** コントローラーにドラッグします。 ポップアップ **メニューで [モーダルに** 表示] を選択します。
1. 追加したセグを選択し、属性インスペクターを **選択します**。 [識別子]**フィールドを** 設定 `userSignedOut` し、[プレゼンテーション] を [**全画面****] に設定します**。

### <a name="create-welcome-page"></a>ウェルカム ページを作成する

1. **Assets.xcassets ファイルを選択** します。
1. [エディター **] メニューで** 、[新しいアセットの **追加] を** 選択し、[イメージ セット] **を選択します**。
1. 新しい **画像アセットを選択** し、 **属性インスペクターを使** って名前を **設定** します `DefaultUserPhoto` 。
1. 既定のユーザー プロファイル写真として使用する画像を追加します。

    ![Xcode のイメージ セット アセット ビューのスクリーンショット](images/add-default-user-photo.png)

1. **GraphTu touchl** **フォルダーに新** しい Cocoa Touch Class ファイルを作成します `WelcomeViewController` 。 フィールド **のサブクラスで UIViewController** **を選択** します。
1. **WelcomeViewController.swift を開き、** その内容を次のコードに置き換えます。

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

1. **Main.storyboard を開きます**。 アイテム **1 シーンを選択し**、ID インスペクター **を選択します**。 クラスの **値を** **WelcomeViewController に変更します**。
1. ライブラリを **使用して**、アイテム 1 シーンに **次の項目を追加します**。

    - 1 **つのイメージ ビュー**
    - 2 **つのラベル**
    - 1 **つのボタン**
1. Connections **Inspector を使用して、** 次の接続を行います。

    - **userDisplayName の出口を** 最初のラベルにリンクします。
    - **userEmail の出口を** 2 番目のラベルにリンクします。
    - **userProfilePhoto の出口を** イメージ ビューにリンクします。
    - signOut **受信アクションを** ボタンの Touch **Up Inside にリンクします**。

1. イメージ ビューを選択し、サイズ検査を **選択します**。
1. 幅と **高** さを 196 に設定します。
1. [配置 **] ボタン** を使用して、 **コンテナー内で水平方向に** 制約を追加し、値を 0 に設定します。
1. [新しい **制約の追加]** ボタン([整列] ボタンの横) を使用して、次の制約を追加します。

    - 上揃え: セーフ エリア、値: 0
    - [下のスペース]: ユーザー表示名、値: Standard
    - 高さ、値: 196
    - 幅、値: 196

    ![Xcode の新しい制約設定のスクリーンショット](./images/add-new-constraints.png)

1. 1 つ目のラベルを選択し、[配置] ボタンを使用して、コンテナー内の水平方向 **の** 制約を値 0 で追加します。
1. [新しい **制約の追加] ボタンを使用** して、次の制約を追加します。

    - Top Space to: User Profile Photo, value: Standard
    - [Bottom Space to]: User Email, value: Standard

1. 2 番目のラベルを選択し、属性インスペクター **を選択します**。
1. 色を **濃い** 灰色 **に変更** し、 **フォント** をシステム **12.0 に変更します**。
1. [配置 **] ボタン** を使用して、 **コンテナー内で水平方向に** 制約を追加し、値を 0 に設定します。
1. [新しい **制約の追加] ボタンを使用** して、次の制約を追加します。

    - [Top Space to]: User Display Name,value: Standard
    - 下のスペース: サインアウト、値: 14

1. ボタンを選択し、属性インスペクタ **ーを選択します**。
1. Title を **次に変更** します `Sign Out` 。
1. [配置 **] ボタン** を使用して、 **コンテナー内で水平方向に** 制約を追加し、値を 0 に設定します。
1. [新しい **制約の追加] ボタンを使用** して、次の制約を追加します。

    - Top Space to: User Email, value: 14

1. シーンの下部にあるタブ バー項目を選択し、[属性インスペクター] を **選択します**。 Title を **次に変更** します `Me` 。

完了すると、ようこそシーンは次のように表示されます。

![ようこそシーンのレイアウトのスクリーンショット](images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a>予定表ページを作成する

1. **GraphTu touchl** **フォルダーに新** しい Cocoa Touch Class ファイルを作成します `CalendarViewController` 。 フィールド **のサブクラスで UIViewController** **を選択** します。
1. **CalendarViewController.swift を開き、** 内容を次のコードに置き換えます。

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

1. **Main.storyboard を開きます**。 アイテム **2 シーンを選択し**、ID インスペクター **を選択します**。 クラスの **値を** **CalendarViewController に変更します**。
1. ライブラリを **使用して**、アイテム 2 **シーンに****テキスト ビューを追加します**。
1. 追加したテキスト ビューを選択します。 [エディター] **メニューで** 、[埋め込み **]、次** に [スクロール ビュー] **の順に選択します**。
1. スクロール ビューとテキスト ビューのサイズを変更して、画面全体を表示します。
1. Connections **Inspector を使用して****、calendarJSON** の出口をテキスト ビューに接続します。
1. シーンの下部にあるタブ バー項目を選択し、[属性インスペクター] を **選択します**。 Title を **次に変更** します `Calendar` 。
1. [エディター **] メニューの**[自動レイアウトの問題の解決] を選択し、予定表ビュー コントローラーのすべてのビューの下にある [不足している制約の追加]**を選択します**。

完了すると、予定表シーンは次のように表示されます。

![カレンダー シーン レイアウトのスクリーンショット](images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a>アクティビティ インジケーターを作成する

1. **GraphTu touchl** **フォルダーに新** しい Cocoa Touch Class ファイルを作成します `SpinnerViewController` 。 フィールド **のサブクラスで UIViewController** **を選択** します。
1. **SpinnerViewController.swift を開き**、内容を次のコードに置き換えます。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SpinnerViewController.swift" id="SpinnerSnippet":::

## <a name="test-the-app"></a>アプリのテスト

変更内容を保存し、アプリを起動します。 [サインイン] ボタンと [サインアウト]ボタンとタブ バーを使用して、画面間を移動できる必要があります。

![アプリケーションのスクリーンショット](images/app-screens.png)
