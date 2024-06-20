# Unity ChatGPT Intergration Demo
This repository demonstrates the integration of ChatGPT with Unity, showcasing how to use ChatGPT's capabilities within a Unity environment to create interactive and intelligent applications.

Other Langs Tutorials
> - [Korean](https://sugar0810.tistory.com/89)
> - [Japanese](https://qiita.com/sugar0810/items/1628ffddcf46c1087a44)

## 🔐 Obtaining an OpenAI API Key
Create an OpenAI account and obtain an API key. You can get the key issued from the [OpenAI API docs](https://platform.openai.com/docs/overview).
1. Login to OpenAI
2. Since registering a card is mandatory, go to the [Billing](https://platform.openai.com/settings/organization/billing/overview) tab and register your card.
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/1577ff03-7581-6de5-c11a-748e8637576b.png)
To prevent excessive charges, go to the Limits tab and set Usage limits.
3. Go to [View API Keys](https://platform.openai.com/api-keys) and create a New Secret Key.
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/f7a21351-27ca-d235-cfbe-1d27f426354b.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/8519ead2-2087-0fbd-7274-53b14ab5c1d9.png)
Once generated, copy the key value (**note that the generated key cannot be re-displayed**).
4. Verify that the key has been generated correctly.
- Check the saved key.
- Verify the supported model (this document uses the "gpt-3.5-turbo" model).

### Windows
1. Verification of cURL Installation
```
curl --version
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/291a329c-0ec1-f813-f4df-1b1cfd3b4434.png)
2. Creating cURL Command
```
curl https://api.openai.com/v1/chat/completions ^
-H "Content-Type: application/json" ^
-H "Authorization: Bearer YOUR_API_KEY" ^
-d "{\"model\": \"gpt-3.5-turbo\", \"messages\": [{\"role\": \"user\", \"content\": \"Say this is a test\"}], \"max_tokens\": 5}"
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/0be0dc65-6462-1e5b-c5e8-45c141f4d514.png)
Enter the key in the underline

### MacOS
```
curl https://api.openai.com/v1/chat/completions \
-H "Content-Type: application/json" \
-H "Authorization: Bearer YOUR_API_KEY" \
-d '{
    "model": "gpt-3.5-turbo",
    "messages": [
        {
            "role": "user",
            "content": "Say this is a test"
        }
    ],
    "max_tokens": 5
}'
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/4a0f0c0a-82d1-a00b-0ce2-a43a67888b09.png)

## ⚙️ Installing Packages and Preparing for Communication
1. Use the package manager to install NuGetForUnity from the URL.
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/edfd3d9c-4c08-12a0-cf1f-b12a9445a6df.png)
2. Search for and install Newtonsoft.Json in the NuGet Packages tab.
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/600e4db1-8b09-61f9-ac63-fc35d1fb0b00.png)

## 📜 Script Creation
```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.Networking;
using Newtonsoft.Json; // Use Newtonsoft.Json

public class OpenAIChatGPT : MonoBehaviour
{
    private string apiKey = "YOUR_API_KEY"; // Replace with an actual API key
    private string apiUrl = "https://api.openai.com/v1/chat/completions";

    public IEnumerator GetChatGPTResponse(string prompt, System.Action<string> callback)
    {
        // Setting OpenAI API Request Data
        var jsonData = new
        {
            model = "gpt-3.5-turbo",
            messages = new[]
            {
                new { role = "user", content = prompt }
            },
            max_tokens = 20
        };

        string jsonString = JsonConvert.SerializeObject(jsonData);

        // HTTP request settings
        UnityWebRequest request = new UnityWebRequest(apiUrl, "POST");
        byte[] bodyRaw = System.Text.Encoding.UTF8.GetBytes(jsonString);
        request.uploadHandler = new UploadHandlerRaw(bodyRaw);
        request.downloadHandler = new DownloadHandlerBuffer();
        request.SetRequestHeader("Content-Type", "application/json");
        request.SetRequestHeader("Authorization", "Bearer " + apiKey);

        yield return request.SendWebRequest();

        if (request.result == UnityWebRequest.Result.ConnectionError || request.result == UnityWebRequest.Result.ProtocolError)
        {
            Debug.LogError("Error: " + request.error);
        }
        else
        {
            var responseText = request.downloadHandler.text;
            Debug.Log("Response: " + responseText);
            // Parse the JSON response to extract the required parts
            var response = JsonConvert.DeserializeObject<OpenAIResponse>(responseText);
            callback(response.choices[0].message.content.Trim());
        }
    }

    public class OpenAIResponse
    {
        public Choice[] choices { get; set; }
    }

    public class Choice
    {
        public Message message { get; set; }
    }

    public class Message
    {
        public string role { get; set; }
        public string content { get; set; }
    }
}
```

```csharp
using UnityEngine;

public class ChatGPTExample : MonoBehaviour
{
    [SerializeField, TextArea(3, 5)] private string prompt;

    void Start()
    {
        OpenAIChatGPT chatGPT = gameObject.AddComponent<OpenAIChatGPT>();
        StartCoroutine(chatGPT.GetChatGPTResponse(prompt, OnResponseReceived));
    }

    void OnResponseReceived(string response)
    {
        Debug.Log("ChatGPT Response: " + response);
    }
}
```

## 📗 Example Usage
Prompt question : In the extended virtual world, what is the newly coined word that combines "meta," which means virtual and transcendent, and "universe," which means world and universe?
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/779a3572-98ba-14fb-7276-dc4471bbd38f.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3748634/0efe936f-e950-f697-f289-102f1be70489.png)

## 🤔 Expected Features
- [Generation Assistance Feature](https://youtu.be/-nnBrHG4mg4?si=mAgEykxHpwSM9btB)
- Educational Quiz Game
- User-Customizable Story Generator
- AR-Based Educational Platform
  - Display 3D models in an AR environment to visualize learning themes
  - Provide real-time Q&A and explanations using ChatGPT
- AR-Based Tour Guide
  - Visually provide information about tourist spots through AR
  - Use ChatGPT to provide additional information about tourist spots and answer questions
- AR-Based Shopping Helper
  - Visually provide product information through AR
  - Use ChatGPT to provide product descriptions and answer questions
