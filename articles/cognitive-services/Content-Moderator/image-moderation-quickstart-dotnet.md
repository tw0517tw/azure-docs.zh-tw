---
title: 快速入門：使用 .NET 來仲裁影像 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 如何使用 Content Moderator SDK for .NET 來仲裁影像
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: sajagtap
ms.openlocfilehash: d89d9b8a2e3b00155e82cc28105007ab39fc549c
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226159"
---
# <a name="quickstart-moderate-images-using-net"></a>快速入門：使用 .NET 來仲裁影像

本文提供資訊和範例程式碼，可協助您開始使用 [Content Moderator SDK for .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 來執行下列操作： 

- 檢查影像中是否有成人或猥褻內容
- 從影像中偵測和擷取文字
- 偵測影像中的人臉

本文假設您已經熟悉 Visual Studio 和 C#。

## <a name="sign-up-for-content-moderator-services"></a>註冊 Content Moderator 服務

您必須有 API 金鑰和 API 帳戶的區域，才能透過 REST API 或 SDK 使用 Content Moderator 服務。
請參考[快速入門](quick-start.md)，以了解如何註冊 Content Moderator 以取得這兩個項目。

## <a name="create-your-visual-studio-project"></a>建立 Visual Studio 專案

1. 將一個新的 [主控台應用程式 (.NET Framework)] 專案新增到您的解決方案。

   在範例程式碼中，將專案命名為 **ImageModeration**。

1. 選取此專案作為解決方案的單一啟始專案。


### <a name="install-required-packages"></a>安裝必要的套件

安裝下列 NuGet 封裝：

- Microsoft.Azure.CognitiveServices.ContentModerator
- Microsoft.Rest.ClientRuntime
- Newtonsoft.Json

### <a name="update-the-programs-using-statements"></a>更新程式的 using 陳述式

修改程式的 using 陳述式。

    using Microsoft.Azure.CognitiveServices.ContentModerator;
    using Microsoft.CognitiveServices.ContentModerator;
    using Microsoft.CognitiveServices.ContentModerator.Models;
    using Newtonsoft.Json;
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Threading;

### <a name="create-the-content-moderator-client"></a>建立 Content Moderator 用戶端

新增下列程式碼，為您的訂用帳戶建立 Content Moderator 用戶端。

> [!IMPORTANT]
> 以您的區域識別碼和訂用帳戶訂用帳戶的值更新 **AzureRegion** 和 **CMSubscriptionKey** 欄位。

    /// <summary>
    /// Wraps the creation and configuration of a Content Moderator client.
    /// </summary>
    /// <remarks>This class library contains insecure code. If you adapt this 
    /// code for use in production, use a secure method of storing and using
    /// your Content Moderator subscription key.</remarks>
    public static class Clients
    {
        /// <summary>
        /// The region/location for your Content Moderator account, 
        /// for example, westus.
        /// </summary>
        private static readonly string AzureRegion = "YOUR API REGION";

        /// <summary>
        /// The base URL fragment for Content Moderator calls.
        /// </summary>
        private static readonly string AzureBaseURL =
            $"https://{AzureRegion}.api.cognitive.microsoft.com";

        /// <summary>
        /// Your Content Moderator subscription key.
        /// </summary>
        private static readonly string CMSubscriptionKey = "YOUR API KEY";

        /// <summary>
        /// Returns a new Content Moderator client for your subscription.
        /// </summary>
        /// <returns>The new client.</returns>
        /// <remarks>The <see cref="ContentModeratorClient"/> is disposable.
        /// When you have finished using the client,
        /// you should dispose of it either directly or indirectly. </remarks>
        public static ContentModeratorClient NewClient()
        {
            // Create and initialize an instance of the Content Moderator API wrapper.
            ContentModeratorClient client = new ContentModeratorClient(new ApiKeyServiceClientCredentials(CMSubscriptionKey));

            client.Endpoint = AzureBaseURL;
            return client;
        }
    }

### <a name="initialize-application-specific-settings"></a>將應用程式特定的設定初始化

將下列靜態欄位新增至 Program.cs 中的 **Program** 類別。

    ///<summary>
    ///The name of the file that contains the image URLs to evaluate.
    ///</summary>
    ///<remarks>You will need to create an input file and update 
    ///this path accordingly. Paths are relative to the execution directory.
    ///</remarks>
    private static string ImageUrlFile = "ImageFiles.txt";

    ///<summary>
    ///The name of the file to contain the output from the evaluation.
    ///</summary>
    ///<remarks>Paths are relative to the execution directory.
    ///</remarks>
    private static string OutputFile = "ModerationOutput.json";


> [!NOTE]
> 範例使用下列影像來產生本快速入門的輸出。
> - https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg
> - https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png

## <a name="store-the-analysis-results"></a>儲存分析結果

將下列類別新增到 **Program** 類別。 使用此類別的執行個體來記錄所檢閱影像的審核結果。

    /// <summary>
    /// Contains the image moderation results for an image, 
    /// including text and face detection results.
    /// </summary>
    public class EvaluationData
    {
        /// <summary>
        /// The URL of the evaluated image.
        /// </summary>
        public string ImageUrl;

        /// <summary>
        /// The image moderation results.
        /// </summary>
        public Evaluate ImageModeration;

        /// <summary>
        /// The text detection results.
        /// </summary>
        public OCR TextDetection;

        /// <summary>
        /// The face detection results;
        /// </summary>
        public FoundFaces FaceDetection;
    }

## <a name="evaluate-an-individual-image"></a>評估個別影像

將下列方法新增至 **Program** 類別。 這個方法會評估單一影像，並傳回評估結果。

> [!NOTE]
> 您的 Content Moderator 服務金鑰會有每秒要求數目 (RPS) 的速率限制。如果超出此限制，SDK 就會擲回錯誤碼為 429 的例外狀況。 
>
> 免費層金鑰有一個 RPS 速率限制。


    /// <summary>
    /// Evaluates an image using the Image Moderation APIs.
    /// </summary>
    /// <param name="client">The Content Moderator API wrapper to use.</param>
    /// <param name="imageUrl">The URL of the image to evaluate.</param>
    /// <returns>Aggregated image moderation results for the image.</returns>
    /// <remarks>This method throttles calls to the API.
    /// Your Content Moderator service key will have a requests per second (RPS)
    /// rate limit, and the SDK will throw an exception with a 429 error code 
    /// if you exceed that limit. A free tier key has a 1 RPS rate limit.
    /// </remarks>
    private static EvaluationData EvaluateImage(
    ContentModeratorClient client, string imageUrl)
    {
        var url = new ImageUrl("URL", imageUrl.Trim());

        var imageData = new EvaluationData();

        imageData.ImageUrl = url.Value;

        // Evaluate for adult and racy content.
        imageData.ImageModeration =
            client.ImageModeration.EvaluateUrlInput("application/json", url, true);
        Thread.Sleep(1000);

        // Detect and extract text.
        imageData.TextDetection =
            client.ImageModeration.OCRUrlInput("eng", "application/json", url, true);
        Thread.Sleep(1000);

        // Detect faces.
        imageData.FaceDetection =
            client.ImageModeration.FindFacesUrlInput("application/json", url, true);
        Thread.Sleep(1000);

        return imageData;
    }

**EvaluateUrlInput** 方法是影像審核 REST API 的包裝函式。
傳回值包含從 API 呼叫所傳回的物件。

**OCRUrlInput** 方法是影像 OCR REST API 的包裝函式。
傳回值包含從 API 呼叫所傳回的物件。

**FindFacesUrlInput** 方法是影像尋找臉部 REST API 的包裝函式。
傳回值包含從 API 呼叫所傳回的物件。

## <a name="process-the-image-urls-in-your-code"></a>處理程式碼中的影像 URL

將以下程式碼新增至 **Main** 方法。

    // Create an object to store the image moderation results.
    List<EvaluationData> evaluationData = new List<EvaluationData>();

    // Create an instance of the Content Moderator API wrapper.
    using (var client = Clients.NewClient())
    {
        // Read image URLs from the input file and evaluate each one.
        using (StreamReader inputReader = new StreamReader(ImageUrlFile))
        {
            while (!inputReader.EndOfStream)
            {
                string line = inputReader.ReadLine().Trim();
                if (line != String.Empty)
                {
                    EvaluationData imageData = EvaluateImage(client, line);
                    evaluationData.Add(imageData);
                }
            }
        }
    }

    // Save the moderation results to a file.
    using (StreamWriter outputWriter = new StreamWriter(OutputFile, false))
    {
        outputWriter.WriteLine(JsonConvert.SerializeObject(
            evaluationData, Formatting.Indented));

        outputWriter.Flush();
        outputWriter.Close();
    }

## <a name="run-the-program-and-review-the-output"></a>執行程式並檢閱輸出

下列 JSON 物件包含程式的輸出。

> [!NOTE]
> `isImageAdultClassified` 代表可能有在特定情況下被視為明顯色情或成人內容的影像存在。
> `isImageRacyClassified` 代表可能有在特定情況下被視為具性暗示或成人內容的影像存在。
>

    [
    {
    "ImageUrl": "https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg",
    "ImageModeration": {
      "cacheID": "7733c303-3b95-4710-a41e-7a322ae81a15_636488005858745661",
      "result": false,
      "trackingId": "WE_e1f20803b4ed471fb5de7df551f5bd9f_ContentModerator.Preview_687c356d-0f00-4aeb-ae5f-c7555af80247",
      "adultClassificationScore": 0.019196987152099609,
      "isImageAdultClassified": false,
      "racyClassificationScore": 0.032390203326940536,
      "isImageRacyClassified": false,
      "advancedInfo": [
        {
          "key": "ImageDownloadTimeInMs",
          "value": "116"
        },
        {
          "key": "ImageSizeInBytes",
          "value": "273405"
        }
      ],
      "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
      }
    },
    "TextDetection": {
      "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
      },
      "metadata": [
        {
          "key": "ImageDownloadTimeInMs",
          "value": "12"
        },
        {
          "key": "ImageSizeInBytes",
          "value": "273405"
        }
      ],
      "trackingId": "WE_e1f20803b4ed471fb5de7df551f5bd9f_ContentModerator.Preview_814fa162-c5d6-4ca6-997b-30ed0686ca83",
      "cacheId": "3fb69496-c64b-4de9-affd-6dd6d23f3e78_636488005876558920",
      "language": "eng",
      "text": "IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n",
      "candidates": []
    },
    "FaceDetection": {
      "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
      },
      "trackingId": "WE_e1f20803b4ed471fb5de7df551f5bd9f_ContentModerator.Preview_a2c40dbe-609d-4eb8-b01c-9988388804ea",
      "cacheId": "e4c0b500-ea8e-4a31-8fb3-35f98c4fbd65_636488005889528303",
      "result": false,
      "count": 0,
      "advancedInfo": [
        {
          "key": "ImageDownloadTimeInMs",
          "value": "11"
        },
        {
          "key": "ImageSizeInBytes",
          "value": "273405"
        }
      ],
      "faces": []
    }
    },
    {
    "ImageUrl": "https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png",
    "ImageModeration": {
      "cacheID": "b4866aa2-5e69-44ed-806a-f9a5d618c8ae_636488005930693926",
      "result": false,
      "trackingId": "WE_e1f20803b4ed471fb5de7df551f5bd9f_ContentModerator.Preview_fdce5510-f689-4791-b081-c2ad54dcfe78",
      "adultClassificationScore": 0.0035635426174849272,
      "isImageAdultClassified": false,
      "racyClassificationScore": 0.021369094029068947,
      "isImageRacyClassified": false,
      "advancedInfo": [
        {
          "key": "ImageDownloadTimeInMs",
          "value": "109"
        },
        {
          "key": "ImageSizeInBytes",
          "value": "2278902"
        }
      ],
      "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
      }
    },
    "TextDetection": {
      "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
      },
      "metadata": [
        {
          "key": "ImageDownloadTimeInMs",
          "value": "46"
        },
        {
          "key": "ImageSizeInBytes",
          "value": "2278902"
        }
      ],
      "trackingId": "WE_e1f20803b4ed471fb5de7df551f5bd9f_ContentModerator.Preview_08a4bc19-6010-41bb-a440-a77278e167d8",
      "cacheId": "28b37471-41b3-4f79-bd23-965498bcff51_636488005950851288",
      "language": "eng",
      "text": "",
      "candidates": []
    },
    "FaceDetection": {
      "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
      },
      "trackingId": "WE_e1f20803b4ed471fb5de7df551f5bd9f_ContentModerator.Preview_40f2ce07-14ba-47cd-ba09-58b557a89854",
      "cacheId": "ec9c1be3-99b7-4bd9-8bc4-dc958c74459f_636488005964914299",
      "result": true,
      "count": 6,
      "advancedInfo": [
        {
          "key": "ImageDownloadTimeInMs",
          "value": "60"
        },
        {
          "key": "ImageSizeInBytes",
          "value": "2278902"
        }
      ],
      "faces": [
        {
          "bottom": 598,
          "left": 44,
          "right": 268,
          "top": 374
        },
        {
          "bottom": 620,
          "left": 308,
          "right": 532,
          "top": 396
        },
        {
          "bottom": 575,
          "left": 594,
          "right": 773,
          "top": 396
        },
        {
          "bottom": 563,
          "left": 812,
          "right": 955,
          "top": 420
        },
        {
          "bottom": 611,
          "left": 972,
          "right": 1151,
          "top": 432
        },
        {
          "bottom": 510,
          "left": 1232,
          "right": 1456,
          "top": 286
        }
      ]
    }
    }
    ]


## <a name="next-steps---get-the-source-code"></a>後續步驟 - 取得原始程式碼

針對這個及其他適用於 .NET 的 Content Moderator 快速入門取得 [Content Moderator .NET SDK](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) 和 [Visual Studio 解決方案](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator)，並開始進行您的整合。
