<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7c08d-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する、反応するネイティブアプリを構築する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="7c08d-101">This tutorial teaches you how to build an React Native app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="7c08d-102">完成したチュートリアルをダウンロードするだけで済む場合は、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-ios-swift)をダウンロードするか、クローンを作成できます。</span><span class="sxs-lookup"><span data-stu-id="7c08d-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-swift).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="7c08d-103">前提条件</span><span class="sxs-lookup"><span data-stu-id="7c08d-103">Prerequisites</span></span>

<span data-ttu-id="7c08d-104">このチュートリアルを開始する前に、開発用のコンピューターに次のものがインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="7c08d-104">Before you start this tutorial, you should have the following installed on your development machine.</span></span>

- [<span data-ttu-id="7c08d-105">Xcode</span><span class="sxs-lookup"><span data-stu-id="7c08d-105">Xcode</span></span>](https://developer.apple.com/xcode/)
- [<span data-ttu-id="7c08d-106">CocoaPods</span><span class="sxs-lookup"><span data-stu-id="7c08d-106">CocoaPods</span></span>](https://cocoapods.org)

> [!NOTE]
> <span data-ttu-id="7c08d-107">このチュートリアルは、Xcode バージョン10.3 と Cocoアポストロフィ Ds バージョン1.7.5 を使用して作成されました。</span><span class="sxs-lookup"><span data-stu-id="7c08d-107">This tutorial was written using Xcode version 10.3 and CocoaPods version 1.7.5.</span></span> <span data-ttu-id="7c08d-108">このガイドの手順は、他のバージョンでは動作しますが、テストされていません。</span><span class="sxs-lookup"><span data-stu-id="7c08d-108">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="7c08d-109">フィードバック</span><span class="sxs-lookup"><span data-stu-id="7c08d-109">Feedback</span></span>

<span data-ttu-id="7c08d-110">このチュートリアルに関するフィードバックは、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-ios-swift)に記入してください。</span><span class="sxs-lookup"><span data-stu-id="7c08d-110">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-ios-swift).</span></span>
